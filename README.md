# Convergence — Client Portal (V2)

A unified client-facing web application for Convergence. This is the **V2 MVP** that merges the standalone TrafficGauge reporting suite into the broader Convergence Portal as a single React-based single-page application.

This repo is intended as the **handoff target** for engineering to convert into a production Vue framework. The MVP is built in React (matching TrafficGauge's existing framework) for design fidelity and rapid iteration, with the explicit understanding that the production conversion will move it to Vue.

---

## Quick Start

This is a **static, zero-build** web app. To run it locally:

```bash
# from the repo root
python3 -m http.server 8000
# then open http://localhost:8000
```

Any static file server works (Node `http-server`, `npx serve`, VS Code Live Server, etc.). React, Babel, Recharts, and PropTypes are all loaded from CDN at runtime — there is **no `npm install` and no build step**.

---

## Deploying

Because this is a pure static site, you can deploy it anywhere that serves files:

- **GitHub Pages** — push to `main`, enable Pages in Settings → Pages, point to `/ (root)`. Done.
- **Vercel** — `vercel deploy` from the repo root. No framework preset needed; pick "Other".
- **Netlify** — drag-and-drop the repo folder, or connect the GitHub repo and leave the build command blank with publish directory `./`.
- **Cloudflare Pages** — same as Netlify; no build, output directory `./`.

---

## Project Structure

```
convergence-portal-v2/
├── index.html              # The entire application (single file)
├── tweaks-panel.jsx        # Reusable Tweaks shell + form-control helpers
├── assets/                 # Place brand logos here (see assets/README.md)
│   └── README.md
├── uploads/                # Optional alternate image directory
├── .gitignore
└── README.md
```

`index.html` and `tweaks-panel.jsx` are the only two source files. Everything else (data, components, styles, theming) lives inside `index.html`.

---

## What's in the App

### Top-level navigation (Convergence sidebar)
1. **Pipelines** — campaign file uploads + scheduled appointment list
2. **TouchPoints** *(default landing screen)* — 3-panel customer view: customer list / timeline / activity summary
3. **TrafficGauge** *(collapsible group)* — the full TrafficGauge reporting suite:
   - Dashboards (library + saved dashboards)
   - Report Builder
   - Data Sources
   - Starred
4. **Portfolio** — sales and service transaction history with KPIs
5. **Weave** — full call log with direction, duration, outcomes, agent

### Persistent sidebar elements
- **Dealership selector** — switch between Metro Automotive Group, Sunrise Honda, etc.
- **Customer card** — name, customer ID, phone, email
- **TrafficGauge group** — auto-expands when any TG sub-screen is active

### Tweaks panel (bottom-right gear icon)
- **Theme:** Electric (default), Neon, Dark Mode
- **Sidebar:** Full / Icons / Hidden
- **Density:** Compact / Default / Spacious
- **Chart Style:** Filled / Outlined / Minimal

---

## Theming & Brand

CSS variables drive all theming. Colors and fonts match the Convergence Brand Guidelines:

| Token | Color | Use |
|-------|-------|-----|
| Electric | `#5E10BC` | Primary accent |
| Neon | `#80E55C` | Success / positive metrics |
| Citrine | `#F7E74B` | Highlight / engagement |
| Jet | `#000000` | Primary text on light backgrounds |
| Lavender | `#D4C9F4` | Secondary accent |
| Purple Mid | `#7E54D9` | Secondary accent |
| Mint | `#ADF792` | Secondary accent |
| Off-White | `#EFEDE8` | Canvas background |

**Fonts:**
- **Rubik** — display / headings
- **Inter Tight** — subheadings, labels, chips
- **Inter** — body copy

All three are loaded from Google Fonts at runtime.

---

## Engineering Handoff: Vue Migration Plan

This MVP is built in **React 18 + Recharts** (CDN, no build step). The intended production stack is **Vue 3** (matching the rest of the Convergence Portal codebase). Suggested migration path:

### 1. Stand up a Vite + Vue 3 project
Use Vue 3 + `<script setup>` with TypeScript. Install Vuetify 3 to match the design system already used in the legacy `Convergence_Portal.html`.

### 2. Port screens one at a time, in this order
The most isolated screens are easiest. Recommended sequence:

1. **PipelinesScreen** → Vue component. ~150 lines of JSX, mostly static.
2. **PortfolioScreen** → Vue component. KPI cards + list, very straightforward.
3. **WeaveScreen** → Vue component. KPI cards + table.
4. **TouchPointsScreen** → Vue component. The most complex of the four — 3-panel layout, customer selection, computed counts/sums from timeline events.
5. **TrafficGauge screens** (`LibraryScreen`, `DashboardScreen`, `ReportBuilderScreen`, `DataSourcesScreen`, `BuilderScreen`) — the largest port. These use Recharts; swap for `vue-chartjs` or `apexcharts` to match the rest of the portal.

### 3. Extract data
All seed data (customers, transactions, calls, campaign stats, etc.) is in `index.html` near the top of the `<script type="text/babel">` block:

- `CX_DEALERS`, `CX_CURRENT_CUSTOMER`
- `CX_APPOINTMENTS_INIT`, `CX_PORTFOLIO_KPIS`, `CX_PORTFOLIO_TXNS`, `CX_WEAVE_KPIS`, `CX_CALLS`
- `CX_CUSTOMERS` (with full timeline per customer)
- `TP_TYPE` (TouchPoints event type → label/icon/color metadata)
- TrafficGauge-specific data: `monthlyLeads`, `inventoryByMake`, etc. starting around line 1525

Move these into a `/src/data/` directory or — for production — replace with API calls to a real backend.

### 4. Theming
The CSS variables defined under `:root`, `[data-theme="sage"]`, and `[data-theme="dark"]` in the `<style>` block in `index.html` are framework-agnostic — copy them verbatim into Vuetify's theme configuration or a global stylesheet.

### 5. Tweaks panel
`tweaks-panel.jsx` is a stylable prototyping shell that listens for `__activate_edit_mode` / `__deactivate_edit_mode` parent-window messages. **This is a prototyping convenience, not a production feature.** In the Vue rewrite, replace it with a proper Vuetify settings dialog or drawer that persists user preferences to `localStorage` or the user profile API.

### 6. Image assets
See `assets/README.md` for the list of brand logo files you need to drop into `/assets/`. The app references them by path (`assets/logo-horizontal.png`); both React and Vue versions can share the same asset folder.

---

## What's Intentionally Missing (Known MVP Limitations)

- **No backend.** All data is hardcoded seed data in `index.html`. Wire to your API during the Vue conversion.
- **No auth.** Add session handling during conversion.
- **No persistence.** State resets on page reload. The Tweaks panel uses `localStorage` for *visual* settings only.
- **No real file uploads.** The Pipelines "Upload File" button captures a filename but does not transmit anything.
- **Brand logos are referenced but not bundled.** See `assets/README.md`.

---

## File Map

| File | Purpose |
|------|---------|
| `index.html` | Entire app: HTML shell, CSS theme, React script, all screen components |
| `tweaks-panel.jsx` | Tweaks panel + form-control helpers (loaded by `index.html`) |
| `assets/README.md` | Brand logo asset checklist |

---

## Credits

V1 source projects merged into V2:
- **Convergence Portal** (Vue 3 + Vuetify) — Pipelines, Portfolio, TouchPoints, Weave
- **TrafficGauge** (React 18 + Recharts) — Dashboards, Report Builder, Data Sources, Starred

Brand guidelines: `Convergence_BrandGuidelines_update.pdf`
