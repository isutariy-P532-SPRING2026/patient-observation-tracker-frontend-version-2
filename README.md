# Patient Observation Tracker — Frontend (Version 2)

[![Pages](https://github.com/isutariy-P532-SPRING2026/patient-observation-tracker-frontend-version-2/actions/workflows/pages/pages-build-deployment/badge.svg)](https://github.com/isutariy-P532-SPRING2026/patient-observation-tracker-frontend-version-2/actions/workflows/pages/pages-build-deployment)

**Live URL:** <https://isutariy-p532-spring2026.github.io/patient-observation-tracker-frontend-version-2/>

**Backend API:** <https://patient-observation-tracker-backend-hg23.onrender.com>

**Backend repo :** <https://github.com/isutariy-P532-SPRING2026/patient-observation-tracker-backend-version-2>

**Frontend repo :** <https://github.com/isutariy-P532-SPRING2026/patient-observation-tracker-frontend-version-2>

Plain HTML + CSS + JavaScript single-page application for the Patient Observation Tracker. No frameworks, no build step — runs entirely in the browser and communicates with the Spring Boot backend via fetch API calls.

---

## Pages

| File | Description |
|------|-------------|

| `index.html` | Patient list — view all patients, add a new patient, switch active user |
| `patient.html?id={n}` | Patient detail — record measurements and category observations, evaluate diagnostic rules, reject observations, undo commands, view full observation history |
| `catalogue.html` | Catalogue management — create phenomenon types (quantitative/qualitative with units and normal ranges), add phenomena to qualitative types, add protocols, create diagnostic rules (conjunctive or weighted) |
| `logs.html` | Logs viewer — command log and audit log side by side, with undo button per command |

---

## Week 2 Changes

### Undo Mechanism

Every state-changing command (record observation, reject observation) can be undone from the Logs page. Clicking **Undo** on a command log entry calls `POST /api/command-log/{id}/undo`, which reverts the observation status and writes an undo entry to the audit log. Undone observations are shown with a purple **UNDONE** badge and hidden from the observation table by default.

### Weighted Diagnostic Rules

The Catalogue page now supports two rule strategies when creating a diagnostic rule:

- **Conjunctive** — fires when all argument observation types are present (Week 1 behaviour)
- **Weighted** — fires when the weighted sum of present argument types meets or exceeds a configurable threshold. Each argument type has an individual weight.

The form shows a threshold field and per-argument weight inputs only when **Weighted** is selected.

### Anomaly Detection

Quantitative phenomenon types now carry `normalMin` and `normalMax` values. When a measurement falls outside this range the backend flags it as anomalous. The observation table shows a dedicated **Anomaly** column with a warning badge for out-of-range readings.

### Multi-User Support

A user switcher in the top-right corner of every page lets the active user be changed between seeded users (admin, alice, bob, staff). The selected username is stored in `sessionStorage` and sent with every mutating request as the `X-Username` header, which the backend uses for command log attribution.

### Category Observation Display Fix

Category observation rows now correctly show:

- **Type column** — the phenomenon type name (e.g. "Blood Group")
- **Value column** — the selected phenomenon name (e.g. "A+")
- **Presence column** — PRESENT or ABSENT

### Pain Level Hierarchy

Pain Level is a qualitative type with a two-level concept hierarchy (None → Any Pain → Mild / Moderate / Severe Pain → Extreme). When a Pain Level observation is recorded, the backend's `PropagationListener` automatically propagates presence/absence up and down the hierarchy.

---

## How to Run Locally

No build step required. Serve with Python to avoid CORS issues:

```bash
python3 -m http.server 3000
# then visit http://localhost:3000
```

To point at a local backend instead of the Render deployment, change the `const API` constant at the top of each page's `<script>` to `http://localhost:8080`.

---

## Key Features

- **Dynamic unit dropdown** — selecting a quantitative phenomenon type auto-populates the unit dropdown from that type's `allowedUnits` (fetched from the API).
- **Dynamic phenomenon dropdown** — selecting a qualitative type fetches and populates its phenomena.
- **Weighted rule form** — threshold and per-argument weight inputs appear dynamically when Weighted strategy is selected.
- **Rule evaluation** — clicking "Evaluate Diagnostic Rules" fires `POST /api/patients/{id}/evaluate` and shows inferred concepts inline.
- **Reject & Undo** — active observations can be rejected with a reason; any command can be undone from the Logs page.
- **Anomaly badges** — out-of-range measurements are flagged with a warning badge in the Anomaly column.
- **Hide filters** — toggles to hide rejected and undone observations from the observation table.
- **Auto-refresh** — observation table refreshes automatically when the browser tab regains focus.

---

## Architecture

All pages follow the same pattern:

``` .
HTML structure (semantic, no framework)
    ↓
Inline <script> with fetch() calls
    ↓
REST API  (https://patient-observation-tracker-backend-hg23.onrender.com/api/...)
```

There is no client-side routing, no state management library, and no bundler. Each page is self-contained.

---

## Deployment

This repository is deployed automatically to **GitHub Pages** from the `main` branch root. Any push to `main` triggers a Pages rebuild.

To deploy your own copy:

1. Fork this repository
2. Go to **Settings → Pages**
3. Set source to `Deploy from a branch` → `main` → `/ (root)`
4. Update the `const API` constant in each HTML file to point to your backend

---

## Related Repository

Backend (Spring Boot): [isutariy-P532-SPRING2026/patient-observation-tracker-backend-version-2](https://github.com/isutariy-P532-SPRING2026/patient-observation-tracker-backend-version-2)
