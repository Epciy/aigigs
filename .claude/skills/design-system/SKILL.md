---
name: design-system
description: AIGigs visual design rules (colors, type, spacing, components) for the React + Tailwind frontend. Use when creating or styling any page or component.
---

# AIGigs design system

Feel: calm, clean, trustworthy. Off-white page, near-black text, lavender as the brand color, soft lavender and cream glows behind sections. Blue and yellow are small accents only.

## Colors (put these in `tailwind.config` under `theme.extend.colors`)
Adjust once, here only, and check contrast (WCAG AA).
| Token | Use | Value |
| --- | --- | --- |
| `lavender` | primary buttons, highlights | `#B99EDE` |
| `lavender-deep` | logo, accents, hover state | `#B492E0` |
| `lavender-soft` | section backgrounds, tags | `#F0EEF8` |
| `cream` | highlight sections, featured gigs | `#F9F6DE` |
| `blue` | links, info, filters | `#4B70E0` |
| `blue-soft` | info badges | `#E8F0F8` |
| `yellow` | tiny accents only (e.g. "New" badge) | `#F6E800` |
| `ink` | headings and main text | `#1A1A17` |
| `body` | body text | `#4E4B40` |
| `muted` | secondary text, placeholders | `#8A897F` |
| `line` | borders | `#E7E5DD` |
| `surface` | page background | `#FAFAFA` |
| `success` / `warning` / `danger` | statuses | `#16A34A` / `#D97706` / `#DC2626` |

Contrast: lavender is light. Text on lavender buttons uses `ink`, not white. Lavender is never used for body text or small text on white.
Glows: soft radial gradients of `lavender-soft` and `cream` are allowed only as large section backgrounds, never inside cards or buttons.

## Type
- Font: Inter (or system sans). Headings 600, body 400.
- Sizes: h1 36/44, h2 28/36, h3 20/28, body 16/24, small 14/20.

## Spacing and shape
- 4 px scale (Tailwind default). Sections `py-16` on desktop, `py-10` on mobile.
- Cards: `rounded-2xl border border-line bg-surface p-5`, light hover shadow.
- Buttons: `rounded-xl px-4 py-2 font-medium`. Primary = `lavender` bg, `ink` text, `lavender-deep` on hover. Secondary = white bg, `line` border, `ink` text. Pill-shaped buttons (`rounded-full`) are fine for hero CTAs.
- Max content width 1120 px.

## Components to build once and reuse
`Button`, `Input`, `Select`, `TagPicker` (skills/languages), `GigCard`, `FilterBar`, `PayBadge`, `StatusBadge`, `EmptyState`, `ErrorState`, `Spinner`, `ChatWidget`, `AdminTable`.

## Rules
- Mobile first. Test at 375 px and 1280 px.
- Every input has a visible label. Visible focus ring. Buttons say what they do ("Apply on Mercor", not "Click here").
- Use tokens only. No new hex values in components.
