# Ledger Loop

A clean, single-file internal dashboard for tracking budgets, invoices, expenses, vendors, and financial approvals.

---

## Project Description

Ledger Loop is a finance workspace built for internal teams who need visibility into where money is allocated and where it's going. It brings budgets, invoices, expenses, vendors, approvals, and reporting into one interface, with live KPIs and charts that update as data changes.

**Core areas:**

| Section | What it does |
|---|---|
| Dashboard | Total/spent/remaining budget, outstanding & overdue invoices, upcoming payments, budget utilization chart, invoice status breakdown, recent activity |
| Budgets | Create and track budgets by department/project, with utilization bars and status (On track / At risk / Over budget) |
| Invoices | Manage invoice lifecycle — Draft → Pending → Approved → Paid, plus Overdue tracking |
| Expenses | Log spend against budgets, categorized, with monthly trend and category breakdown charts |
| Vendors | Vendor directory with total invoiced and outstanding balance per vendor |
| Approvals | Queue of pending invoices/expenses awaiting sign-off, with approve/reject actions |
| Reports & Analytics | Budget vs. actual, invoice aging, vendor spending, cash flow overview |
| Team & Permissions | Manage internal users and roles (Admin, Finance Manager, Approver, Team Member, Viewer) |
| Settings | Company profile, currency & tax, invoice preferences, notifications, integrations, security |

**Tech stack:** a single self-contained `index.html` file — HTML, CSS (custom properties for theming), vanilla JavaScript, and [Chart.js](https://www.chartjs.org/) (loaded via CDN) for charts. No build step, no backend, no database.

**Data:** the app ships with realistic mock data (budgets, invoices, expenses, vendors, team members) held in an in-memory JavaScript object. Nothing is persisted — refreshing the page resets the app to its seeded state. This is intentional for a demo/prototype; see [Known Issues](#known-issues) for what that means in practice.

---

## Diagram

```
┌─────────────────────────────────────────────────────────┐
│                        Browser tab                      │
│  ┌───────────┐  ┌──────────────────────────────────┐    │
│  │  Sidebar  │  │              Topbar               │    │
│  │           │  ├──────────────────────────────────┤    │
│  │ Dashboard │  │                                    │    │
│  │ Budgets   │  │         Active page content        │    │
│  │ Invoices  │  │   (Dashboard / Budgets / Invoices  │    │
│  │ Expenses  │  │    / Expenses / Vendors / etc.)    │    │
│  │ Vendors   │  │                                    │    │
│  │ Approvals │  │   cards · tables · Chart.js charts │    │
│  │ Reports   │  │                                    │    │
│  │ Team      │  └──────────────────────────────────┘    │
│  │ Settings  │                                            │
│  └───────────┘                                            │
└─────────────────────────────────────────────────────────┘
        │
        ▼
  state = { budgets, invoices, expenses, vendors,
            approvals, team, notifications, activity }
        │
        ▼
  All reads/writes happen in-memory (JS object) —
  no server, no database, no localStorage.
```

**File structure:**

```
ledger-loop/
└── index.html   ← everything: markup, styles, mock data, app logic
```

*(No screenshots are included in this README — open `index.html` in a browser to see the live UI.)*

---

## Installation Instructions (for users)

Ledger Loop requires no installation, build tools, or server.

1. Download `index.html`.
2. Double-click it, or open it via your browser's **File → Open** menu.
3. That's it — the app loads fully in the browser tab.

**Requirements:**
- A modern browser (Chrome, Firefox, Safari, or Edge — anything released in the last few years).
- An internet connection on first load, since fonts (Google Fonts) and Chart.js are loaded from a CDN. Once loaded, most interactions work offline until you refresh.

**Optional — serve it locally instead of opening the file directly:**

```bash
# from the folder containing index.html
python3 -m http.server 8000
# then visit http://localhost:8000 in your browser
```

This avoids any browser quirks around the `file://` protocol and is closer to how it'd behave once deployed.

---

## Instructions for Contributors (Developers)

There's no package manager, framework, or build pipeline — the entire app is one HTML file, which keeps the barrier to contributing low.

**Getting set up:**

1. Clone or copy the project folder.
2. Open `index.html` in your editor of choice (VS Code, etc.).
3. Open the same file in a browser to preview changes. Use a local server (see above) or your editor's live-reload extension (e.g. VS Code's "Live Server") for a faster feedback loop than manually refreshing.

**Code layout inside `index.html`:**

- `<style>` block — design tokens live at the top under `:root` (`--navy`, `--emerald`, `--bg`, `--text`, etc.), followed by component styles (sidebar, cards, tables, modals, charts, settings).
- `<body>` — static markup for the shell (sidebar, topbar), then one `<section class="page" id="page-*">` per page, then modal markup at the bottom.
- `<script>` block, roughly top to bottom:
  - Formatting helpers (`fmt`, `fmtShort`, date helpers)
  - `state` — the single source of truth (all mock data)
  - Navigation (`navItems`, `goToPage`, `renderNav`)
  - Per-page render functions (`renderDashboard`, `renderBudgets`, `renderInvoices`, etc.) — each one reads from `state` and re-renders its DOM section
  - Modal open/submit functions (`openInvoiceModal` / `submitInvoice`, etc.) — these mutate `state` and then call the relevant render function
  - `init()` at the very bottom, which calls every render function once on load

**Conventions to follow:**

- Any function that changes `state` should call the render function(s) for whatever it affects, and often `renderDashboard()` too, since KPIs are derived from everything else.
- Keep new UI consistent with the existing design tokens (CSS variables) rather than hardcoding new colors.
- Charts are destroyed and recreated on every render (`if(chart) chart.destroy()`) rather than updated in place — follow that pattern for new charts to avoid duplicate canvases.
- No `localStorage`/`sessionStorage` is used anywhere — keep it that way unless you're deliberately adding real persistence (see below).

**Testing changes:** there's no test suite. Manually click through the flows you touched (add/approve/edit/delete) and check the browser console for errors.

---

## How to Tweak This Project for Your Own Use

**Change the branding:**
- Company name / logo: edit `.sidebar-head` in the HTML and the `loop-mark` SVG.
- Colors: everything flows from the CSS custom properties in `:root` at the top of the `<style>` block — change `--navy` and `--emerald` to your brand colors and the whole UI updates.
- Fonts: swap the Google Fonts `<link>` and the `--font-ui` / `--font-num` variables.

**Change the currency:**
- The `fmt()` function near the top of the `<script>` block controls how amounts are displayed (currently `$` with US formatting). Change the symbol and/or the `toLocaleString` locale there.

**Replace the mock data with your own:**
- Edit the arrays inside the `state` object (`state.budgets`, `state.invoices`, `state.expenses`, `state.vendors`, `state.team`, etc.) directly, or wire up a fetch call in `init()` to load real data before the first render.

**Add a new page:**
1. Add an entry to `navItems` and `pageTitles`/`quickAddMap`.
2. Add a new `<section class="page" id="page-yourpage">` in the body.
3. Write a `renderYourPage()` function and call it from `init()`.

**Add real persistence:**
- The app currently keeps everything in memory only. To persist data, you'd connect `state` to a backend (REST API, Firebase, Supabase, etc.) — load data into `state` on `init()`, and save on every mutation (inside the `submit*` functions and action handlers like `approveInvoice`, `markPaid`, `decideApproval`).
- If you split this into a real application, `state` is the natural boundary to replace with API calls.

---

## Expectations from Contributors

- **Keep it dependency-light.** The whole point of this project is that it's a single file with no build step. Please don't introduce a framework, bundler, or package manager without discussing it first.
- **Match the existing visual language.** Use the established design tokens, spacing, and component patterns (cards, pills, mini-buttons) rather than introducing new one-off styles.
- **Don't break the "no persistence" contract silently.** If you add `localStorage`, an API, or any other persistence layer, document it clearly in this README so users understand their data is no longer ephemeral.
- **Test your changes by hand** across at least Chrome and Firefox before submitting, since there's no automated test suite to catch regressions.
- **Keep accessibility in mind** — maintain focus states, keep interactive elements keyboard-reachable, and don't remove the `prefers-reduced-motion` handling.
- **Be descriptive in commit messages / PRs** about which page(s) or function(s) you touched, since there's no file-level separation to signal scope at a glance.

---

## Known Issues

- **No data persistence.** All data lives in memory. Refreshing or closing the tab resets everything to the seeded mock data. Anything added, edited, or approved during a session is lost.
- **No authentication.** There is no login, session, or user-switching — the app always presents the same signed-in user ("Dana Kimathi") regardless of who's using it.
- **No real backend / API.** Actions like "Upload invoice," "Export report," and "Export budget data" show a confirmation toast but don't actually generate or transfer a file.
- **Requires internet on first load.** Google Fonts and Chart.js are pulled from a CDN; without connectivity on first load, the app will fall back to system fonts and charts will fail to render.
- **Single-user only.** There's no concept of multiple people using the app simultaneously — no real-time sync, no conflict handling.
- **Mobile layout is functional but not fully optimized.** The sidebar collapses behind a menu button below ~860px width and tables scroll horizontally, but dense tables (e.g. Invoices, Budgets) are still tight on very small screens.
- **Approvals and invoice/expense status aren't linked.** Approving an item in the Approvals queue doesn't automatically update the status of the corresponding invoice/expense in their own list, and vice versa — they're separate mock datasets rather than one linked source of truth.
- **No form validation beyond required-field checks.** Numeric fields don't guard against negative numbers, and dates aren't validated against each other (e.g. a due date before an issue date is accepted).
