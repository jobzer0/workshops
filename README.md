# JOBZER0 Workshops

A [Fumadocs](https://fumadocs.dev) site for hosting workshop content, deployed on Cloudflare via OpenNext.

## Quick Start

```bash
npm install
npm run dev
```

Open http://localhost:3000 — the landing page links to `/workshops`.

## Project Structure

| Path | Purpose |
|------|---------|
| `content/workshops/` | MDX workshop content (each subfolder is a workshop) |
| `source.config.ts` | Fumadocs MDX config — points at `content/workshops` |
| `src/app/workshops/` | Next.js app router pages for the `/workshops` route |
| `src/app/og/workshops/` | OG image generation for workshop pages |
| `src/app/llms.mdx/workshops/` | LLM-readable markdown route |
| `src/app/(home)/` | Landing page |
| `src/app/api/search/` | Search route handler |
| `src/lib/shared.ts` | Central config (app name, routes, git config) |
| `src/lib/layout.shared.tsx` | Nav layout options (title, GitHub link) |
| `src/css/jobzero.css` | Custom color theme |

## Customizations from Fumadocs Defaults

Below is a summary of every customization made to the default Fumadocs scaffold. If a Fumadocs upgrade breaks something, check these files first.

---

### 1. Route Path: `/docs` → `/workshops`

The default `/docs` route was renamed to `/workshops` across the entire project.

**Files changed:**

| File | What changed |
|------|-------------|
| `src/lib/shared.ts` | `docsRoute`, `docsImageRoute`, `docsContentRoute` all use `/workshops` |
| `source.config.ts` | `dir` changed from `'content/docs'` to `'content/workshops'` |
| `src/app/workshops/[[...slug]]/page.tsx` | Route type generics use `'/workshops/[[...slug]]'` |
| `src/app/workshops/layout.tsx` | Layout type uses `'/workshops'` |
| `src/app/og/workshops/[...slug]/route.tsx` | Route type uses `'/og/workshops/[...slug]'` |
| `src/app/llms.mdx/workshops/[[...slug]]/route.ts` | Route type uses `'/llms.mdx/workshops/[[...slug]]'` |
| `src/app/(home)/page.tsx` | Link text and href updated to `/workshops` |
| `middleware.ts` | No changes needed — reads from `shared.ts` dynamically |

**Directories renamed:**

- `src/app/docs/` → `src/app/workshops/`
- `src/app/og/docs/` → `src/app/og/workshops/`
- `src/app/llms.mdx/docs/` → `src/app/llms.mdx/workshops/`
- `content/docs/` → `content/workshops/`

**If Fumadocs regenerates types:** The `PageProps<>` and `RouteContext<>` generics are derived from the filesystem route. If these folders are renamed or Fumadocs changes how types are generated, update the type strings in `page.tsx`, `layout.tsx`, and `route.ts`/`route.tsx` to match the new folder paths.

---

### 2. GitHub Link

**File:** `src/lib/shared.ts`

```typescript
export const gitConfig = {
  user: 'jobzer0',
  repo: 'workshops',
  branch: 'main',
};
```

This feeds into:
- The GitHub icon in the nav bar (`src/lib/layout.shared.tsx`)
- The "Edit on GitHub" link on each page (`src/app/workshops/[[...slug]]/page.tsx`)

---

### 3. Custom Theme: `jobzero.css`

**File:** `src/css/jobzero.css`

Based on the Fumadocs `black` preset (`fumadocs-ui/css/black.css`) with the primary/highlight color changed to an orange derived from `#FF9900`.

**Key design decisions:**
- **Dark mode primary:** `hsl(36, 100%, 50%)` = `#FF9900` (full AWS orange, 7.2:1 contrast on black)
- **Light mode primary:** `hsl(32, 100%, 40%)` = `#CC7A00` (darkened orange, ~4.6:1 contrast on white, WCAG AA compliant)
- **Primary foreground:** White text in light mode, black text in dark mode (for contrast on the orange)
- **Ring color:** Matches primary in each mode

**How it's loaded (`src/app/global.css`):**

```css
@import 'tailwindcss';
@import '../css/jobzero.css';
@import 'fumadocs-ui/css/preset.css';
```

The import order matters: `jobzero.css` replaces the default theme (e.g., `fumadocs-ui/css/neutral.css`), then `preset.css` adds the Fumadocs utilities on top.

**If Fumadocs changes their theme variables:** Check `node_modules/fumadocs-ui/css/black.css` for new or renamed `--color-fd-*` variables. Our file imports `fumadocs-ui/css/lib/default-colors.css` for base values — if that path changes, update the `@import` in `jobzero.css`.

---

### 4. Styled Nav Title (JOBZER0 with colored "0")

**File:** `src/lib/layout.shared.tsx`

```tsx
nav: {
  title: (
    <span>JOBZER<span className="text-fd-primary">0</span></span>
  ),
},
```

The `text-fd-primary` utility class uses the `--color-fd-primary` CSS variable from our theme, so the "0" is orange and automatically adapts to light/dark mode.

**Important notes:**
- The `appName` export in `shared.ts` (`'JOBZER0'`) is still used for the OG image generator (`src/app/og/workshops/[...slug]/route.tsx`) as plain text. The styled version is only in the nav.
- JSX whitespace matters — the text must be on a single line (`JOBZER<span>0</span>`) with no spaces/newlines between elements, otherwise a visible gap appears.
- If Fumadocs changes the `nav.title` prop type or the `text-fd-primary` utility class name, this will break. Check [fumadocs.dev/docs/ui/theme](https://fumadocs.dev/docs/ui/theme) for current utility prefixes.

---

### 5. Content Structure

Workshop content lives in `content/workshops/`. Each workshop is a subfolder with its own `meta.json` for sidebar navigation:

```
content/workshops/
├── meta.json              # Root nav (lists workshop folders)
├── index.mdx              # Landing page for /workshops
├── test.mdx               # Sample page (can be removed)
└── prowler-cli/           # First workshop
    ├── meta.json          # Page ordering for sidebar
    ├── index.mdx          # Workshop overview
    ├── prerequisites.mdx
    ├── installation.mdx
    ├── first-scan.mdx
    ├── output-formats.mdx
    ├── dashboard.mdx
    ├── ec2-deployment.mdx
    ├── cloudformation.mdx
    ├── cli-reference.mdx
    └── troubleshooting.mdx
```

To add a new workshop: create a new subfolder under `content/workshops/`, add a `meta.json` with page ordering, then add it to the root `content/workshops/meta.json` pages array.

---

## Troubleshooting After Fumadocs Upgrades

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| Build fails with type errors in `page.tsx` or `route.ts` | Route type generics changed | Update `PageProps<>` / `RouteContext<>` strings to match current folder paths |
| Theme colors revert to defaults | `fumadocs-ui/css/lib/default-colors.css` path changed | Check `node_modules/fumadocs-ui/css/` for new structure, update import in `jobzero.css` |
| `text-fd-primary` class not found | Utility prefix changed | Check fumadocs theme docs for current prefix (was `fd-` as of v16) |
| Nav title shows unstyled text | `nav.title` prop type changed | Check `BaseLayoutProps` type definition for current nav options |
| Content not loading | `defineDocs` API changed | Check `source.config.ts` against current fumadocs-mdx docs |
| Middleware errors | Fumadocs negotiation API changed | Check `fumadocs-core/negotiation` exports |

## Build & Deploy

```bash
# Local development
npm run dev

# Production build (Next.js)
npm run build

# Deploy to Cloudflare (via OpenNext)
npm run deploy
```

## Learn More

- [Fumadocs Documentation](https://fumadocs.dev)
- [Fumadocs Theme Reference](https://fumadocs.dev/docs/ui/theme)
- [Next.js Documentation](https://nextjs.org/docs)
- [OpenNext for Cloudflare](https://opennext.js.org/cloudflare)
