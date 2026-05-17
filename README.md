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

> Note: open the app through a local server (the `http.server` command above), not by double-clicking `index.html`. Opening it as a `file://` URL will block the CDN scripts and the app will render a blank page.

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
├── tweaks-panel.jsx        # Reference copy of the Tweaks shell (already inlined in index.html)
├── assets/                 # Place brand logos here (see assets/README.md)
│   └── README.md
├── uploads/                # Optional alternate image directory
├── .gitignore
└── README.md
```

`index.html` is fully self-contained — it is the only file required to run the app. `tweaks-panel.jsx` is a **reference copy** of the Tweaks panel module that is already inlined inside `index.html`; it is kept as a separate file so engineering can diff/extract it cleanly during the Vue migration. Editing it does **not** affect the running app unless you also update the inlined copy in `index.html`.

---

## What's in the App

### Top-level navigation (Convergence sidebar)
1. **Pipelines** — campaign file uploads + an **Integrations** tab (see below)
2. **TouchPoints** — 3-panel customer view: customer list / timeline / activity summary
3. **TrafficGauge** *(collapsible group)* — the full TrafficGauge reporting suite:
   - Dashboards (library + saved dashboards)
   - Report Builder
   - Data Sources
   - Starred
4. **Portfolio** — sales and service transaction history with KPIs
5. **Weave** — full call log with direction, duration, outcomes, agent

### Persistent sidebar elements
- **Dealership selector** — switch between Metro Automotive Group, Sunrise Honda, etc.
- **Customer card** — current customer context
- **Style / Tweaks panel** — runtime theming and layout controls (prototyping tool)

### New in V2: Pipelines → Integrations
The **Pipelines** screen now has two tabs: **Files** and **Integrations**.

The Integrations tab shows a card grid of available connectors:

- **Inventory** — *"Bring in your inventory right from your website"* — **active / wired**
- DMS Sync, CRM Connector, Website Tracking, Email Platform, Service Scheduler — *"Coming soon"* placeholders so the screen feels populated

Clicking the **Inventory** card connects it: a new **Inventory** data source is added under **TrafficGauge → Data Sources**, and the app jumps you straight there so you can browse the table just like Calendar Events, Phone Calls, etc. Connected integrations switch to a **"Connected"** badge with a **"View data"** action.

**Connection state persists in `localStorage`** under the key `cx_integrations` (an array of enabled integration ids). Clearing site data or using a different browser resets it.

---

## State & Persistence (localStorage keys)

This MVP persists a few things client-side via `localStorage`. Engineering should replace each of these with server-side persistence in the Vue version:

| Key                 | Purpose                                                      |
|---------------------|--------------------------------------------------------------|
| `cx_integrations`   | Array of enabled integration ids (Pipelines → Integrations)  |
| `tg_widget_layouts` | Per-dashboard widget layout positions                        |
| `tg_*` (tweaks)     | Tweaks-panel theme / density / chart-style preferences        |

---

## Engineering Handoff — Vue Migration Notes

The MVP is intentionally built in React to match the existing TrafficGauge codebase and preserve design fidelity. The production target is Vue. Suggested approach:

1. **Stand up the Vue shell first** — sidebar, routing, theming tokens (the CSS custom properties in the `:root` block of `index.html` are framework-agnostic and can be lifted directly).
2. **Port the simplest screens first**, in this order: Pipelines → Portfolio → Weave → TouchPoints. These have minimal interactivity and shared data shapes.
3. **Port TrafficGauge last** — it is the largest and most stateful surface (Report Builder, Dashboards, Data Sources). The Report Builder's chart rendering uses Recharts; pick a Vue charting equivalent (e.g. `vue-chartjs` or ECharts) early so the data adapters are written once.
4. **Replace all seed data with API calls.** The seed data is hardcoded near the top of the script block in `index.html`. Search for these identifiers:
   - `CX_DEALERS`, `CX_CURRENT_CUSTOMER`, `CX_APPOINTMENTS_INIT`, `CX_PORTFOLIO_KPIS` — Convergence Portal data
   - `DATABASES`, `INVENTORY_DATASOURCE` — TrafficGauge data sources
   - `INTEGRATIONS_CATALOG` — the Integrations card grid (only `inventory` is wired; the rest are `comingSoon` placeholders)
5. **Replace `localStorage` with real persistence** for the keys listed in the table above.
6. **The Tweaks / Style panel is a prototyping tool, not a production feature.** In the Vue rewrite, replace it with a real settings UI (or remove it entirely). `tweaks-panel.jsx` is the isolated reference for what it does.

---

## Notes

- All seed data (dealership names, customer names, phone numbers, transaction history) is **fictional demo data** carried over from the original prototype files. Review it before making the repo or any deployment public.
- The sidebar logo loads from `assets/logo-horizontal.png`. If that file is absent, the app falls back gracefully to a text/"C" mark — see `assets/README.md`.
