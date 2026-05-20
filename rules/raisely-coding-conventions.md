---
description: Coding conventions for Raisely custom components - React patterns, CSS, theming, and lifecycle
alwaysApply: true
---

# Raisely custom component conventions

These conventions apply to **custom components** authored by users (the ones that live in a campaign's `components/` directory and ship via the CLI). They do not apply to core blocks or platform internals.

For anything not covered here, read the official docs first via the Raisely MCP. Do not re-derive what the docs already specify.

- Custom components overview: `raisely-docs://guides/introduction`
- Component reference and helpers: `raisely-docs://guides/components/README`
- Block / component registry: `raisely-docs://index`
- Page JSON shape (where components are placed): `raisely-docs://guides/page-json-format`
- Core blocks (for naming, prop, and behaviour parity): `raisely-docs://guides/core-blocks`
- CLI workflow for custom components: `raisely-docs://guides/cli`

If a convention below conflicts with the docs, the docs win - update or remove the convention here.

## File layout

- One component per file. File name and exported component name match (`MyDonationCta.js`, `export default class MyDonationCta`).
- Keep helpers and small subcomponents in the same file unless they are reused. If reused, factor into a shared file under the same `components/` tree.
- Co-locate component-specific styles with the component file, following the structure documented in `raisely-docs://guides/components/README`.

## React patterns

- Match the component shape shown in the docs (props, lifecycle, exposed helpers). Don't introduce patterns the docs don't cover.
- Use the helpers Raisely exposes (e.g. data hooks, donation form helpers, profile/user helpers) instead of re-fetching or hand-rolling state. Look them up in `raisely-docs://guides/components/README` rather than guessing the API.
- Keep components presentational where possible; push data fetching and side effects to the helpers/hooks Raisely provides.
- Treat props as the only stable contract. Don't reach into platform internals or undocumented globals.
- No top-level side effects on import. All work happens inside the component lifecycle.
- Prefer small, composable components over one large component with many conditional branches.

## CSS conventions

- Scope styles to the component. Use the styling approach the docs prescribe (CSS-in-JS or scoped class names per the docs); don't introduce a new styling system.
- Never write global selectors (`body`, `*`, raw tag selectors) from a custom component. They leak across the whole campaign.
- Use design tokens from the campaign theme (colors, spacing, typography) instead of hard-coded values. See the theming section below.
- Keep breakpoints consistent with the campaign's theme breakpoints; don't invent new ones.
- Avoid `!important`. If you need it, the selector is too weak or you're fighting the theme - fix the cause.

## Theming

- Pull colors, fonts, spacing, and radii from the active theme, not literals. The theme is the single source of truth for visual identity across pages.
- Components must render correctly under any theme variant the campaign supports. Test against the campaign's actual theme in `raisely local` before deploying.
- For values not in the theme, expose a component prop (configurable in the editor) instead of hard-coding.

## Component lifecycle and editor behaviour

- Components must render in the **editor** as well as the live site. Avoid code paths that assume a real DOM, real network, or a real user (e.g. `window.location` parsing, auth-required fetches without a fallback).
- Guard against missing data: a profile, user, or donation may be `null` or `undefined` while loading. Render a sensible empty state.
- Server-side rendering: assume the component may be rendered on the server. Guard `window`/`document` access with the patterns shown in `raisely-docs://guides/components/README`.
- Hot reload: edits to a component file should not require a campaign restart. If you find yourself adding module-level singletons or caches, reconsider.

## Configurable props

- Every value the campaign editor might want to change should be a prop, not a constant. Document props in the component header per the docs.
- Default props must produce a render that works out of the box on a fresh campaign.
- Keep prop names aligned with core block conventions (`title`, `description`, `theme`, `style`, etc.) so editor users have a predictable experience. Cross-check against `raisely-docs://guides/core-blocks`.

## Data and side effects

- Use Raisely's data helpers for donations, profiles, users, events, and campaign data. Don't call the public API directly from a custom component unless the docs say to.
- Cache and pagination are handled by the helpers; don't add a parallel cache.
- Forms (donation, signup, custom): extend or compose with the documented form helpers; don't reimplement validation, payment handling, or submission logic.

## Don'ts

- Don't import from undocumented internal paths.
- Don't ship secrets, API keys, or environment-specific URLs in a custom component. Use props or campaign settings.
- Don't bypass the form/payment helpers to talk to Stripe or any processor directly. See `raisely-guardrails.md`.
- Don't add analytics or tracking calls inside a component without confirming with the user; analytics belong in campaign-level config.
- Don't introduce a new dependency for something a Raisely helper already provides.

## Workflow

1. Edit the component file under the campaign's `components/` tree.
2. Verify in `raisely local` (see `raisely-tool-usage.md` and the `raisely-start-local` skill).
3. Take a screenshot of the change in the preview.
4. Summarize the diff for the user (per `raisely-guardrails.md`).
5. Deploy with `raisely deploy` only after explicit approval.
