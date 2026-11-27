# `@alpha/ui`

Shared design package for Alpha apps (Admin, Player, marketing sites).

Alpha = private Drive + Mail + Chat on your domain. This package owns **chrome only** — not Drive/session/chat/mail domain logic.

## Owns / does not

| Owns | Does not |
|------|----------|
| CSS tokens (`tokens/tokens.css`) | Cryptree / session / JMAP |
| SVG icons + Svelte icon wrappers | Product routing / copy |
| Shell, sheets, popovers, composers | Space-specific business rules |

## Install

```json
{ "dependencies": { "@alpha/ui": "file:../alpha-ui" } }
```

Vite: `ssr: { noExternal: ['@alpha/ui'] }`.

```css
@import '@alpha/ui/tokens.css';
@import '@alpha/ui/popover.css';
@import '@alpha/ui/scrollbars.css'; /* mobile: hide scrollbars; opt-in `.show-scrollbar` */
```

Override Space brand knobs (`--accent`, fonts) on `<html>` — don’t fork token values in app `:root`. See `alpha-docs/ADR-shared-design-system.md`.

## Key exports

| Export | Role |
|--------|------|
| `UiShell.svelte` | App frame: nav, list column, mobile pill, detail slot |
| `DetailChrome.svelte` / `ChromeMobileBar.svelte` | Detail frames |
| `Modal.svelte` / `CreateSheet.svelte` | Sheets (no drag grabber) |
| `FloatPanel.svelte` / `DropdownMenu.svelte` | Popovers — `.alpha-popover-*`; anchor or `{x,y}` point |
| `ChatBubble.svelte` / `ChatComposer.svelte` | Presentational chat chrome |
| `Button.svelte` / `PrimaryButton.svelte` / `GlassButton.svelte` | CTAs |
| `NavIcon` / `icons/*` | SVG SoT in `icons/*.svg` |
| `scrollbars.css` | `data-h-scroll` = horizontal row beats page swipe |

## Session cookies

Admin and Player share Drive auth via parent-domain cookie `Domain=.{apex}` (e.g. `.example.com`) so deep-links don’t re-login.

## Design rules (short)

- Accent is a **whisper** (checks / sparse marks) — not selected rows or primary CTAs.
- Selected list rows: `--fg-4` fill, not accent wash.
- No monospace / tabular-nums in chrome counts.
- Destructive icons: `--accent-3`; in-progress: `--accent-2`; Settings disconnect: grayscale.
