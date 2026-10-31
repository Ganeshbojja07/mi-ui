# mi-ui

React dashboard for the motor insurance automation: case list, case detail with documents and extracted fields, stage timeline, email thread, and reviewer approve/reject. It talks only to `../mi-backend`'s REST API. See `../CLAUDE.md` for the full picture.

## Stack

React 19 · TypeScript (strict) · Vite · Tailwind CSS v4 (CSS-first `@theme`) · TanStack Query · React Router · Motion (`motion/react`) · `openapi-typescript` + `openapi-fetch` · react-pdf or image viewer for previews · Vitest + Testing Library · MSW for API mocks · Playwright for e2e · ESLint + Prettier.

## Commands

```bash
npm install
npm run dev          # http://localhost:5173, proxies /api → http://localhost:8080
npm run build
npm run test         # vitest
npm run test:e2e     # playwright (backend or MSW required)
npm run lint && npm run typecheck
npm run gen:api      # regenerate src/api/schema.d.ts from ../mi-backend/contracts/openapi.yaml
```

## Layout

```
src/
  api/          # schema.d.ts (GENERATED), client.ts (openapi-fetch), query hooks per resource
  components/   # Button, StatusBadge, StatCard, Table, Drawer, Toast, Skeleton, Timeline, ConfidenceField
  features/
    dashboard/
    cases/      # list, filters, search by Case ID or email
    case-detail/# documents viewer, fields, rule results per round, AI comments, emails, timeline, review bar
  layouts/      # AppShell with blue header nav
  lib/          # formatters (dates, masked EID), motion presets
  mocks/        # MSW handlers + fixtures
  index.css     # Tailwind import + @theme tokens
```

## Theme (must match these tokens)

```css
@import "tailwindcss";
@theme {
  --font-sans: "Inter", ui-sans-serif, system-ui, sans-serif;
  --color-brand-50:  #EEF3FC;
  --color-brand-600: #1047B9;
  --color-brand-700: #0C3A99;
  --color-navy-900:  #252B52;
  --color-ink:       #141414;
  --color-canvas:    #F8F8F8;
  --color-accent:    #2ED3A0;
  --radius-btn: 2px;
  --ease-smooth: cubic-bezier(0.22, 1, 0.36, 1);
}
```

Look: royal-blue full-width header with white semibold nav links, near-white canvas, large light-weight headings (`font-light tracking-tight`), stat blocks with big light numbers and a `border-b-4 border-brand-600` rule, square-ish buttons, navy panels for the review bar and toasts. Borrow the style only — no third-party logos or brand names.

Status badges: APPROVED emerald · PROCESSING brand-50/brand-700 with pulsing dot · AWAITING_APPLICANT amber · NEEDS_REVIEW violet · REJECTED/FAILED red.

## Conventions

- All server data via TanStack Query hooks in `src/api/`; query keys `['cases', filters]`, `['case', caseId]`, `['case', caseId, 'stages', round]`. Mutations invalidate affected keys. Poll active cases every 5 s.
- Types come from the generated schema only; never hand-write API types.
- Every interactive element: 150–250 ms transitions with `ease-smooth`, hover lift, press scale, visible `focus-visible` ring in `accent`, and `motion-reduce:` fallbacks. Animate only `transform`/`opacity`.
- Motion for enter/exit: page fade, case drawer slide-in, row reordering (`layout`), toasts. Respect `useReducedMotion()`.
- Loading = skeletons; button shows an inline spinner without changing size; top `accent` progress bar on navigation.
- Low-confidence fields (< 0.85) are highlighted; Emirates ID numbers are displayed masked.
- Accessible: semantic HTML, labels, keyboard navigation in tables and drawer, contrast AA.

## Testing expectations

- Component tests for Button, StatusBadge, Timeline, ConfidenceField.
- Feature tests with MSW for list filtering, case detail rendering, and review approve/reject.
- One Playwright happy-path test: open dashboard → open case → approve from review bar.

## Don'ts

- Don't call Temporal, MinIO or the Python worker directly — only `/api/*`.
- Don't hand-edit `src/api/schema.d.ts`.
- Don't add a second styling system (no CSS-in-JS, no component kit that fights Tailwind).
