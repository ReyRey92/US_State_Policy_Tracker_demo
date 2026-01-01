# US State Policy Tracker (Typesense + InstantSearch)

A static, browser-based search interface for exploring a curated dataset of US state policy / legislative items across all 50 states, territories, and special sessions.
It uses **Typesense** as the search backend and **InstantSearch.js** for a fast UI with faceted filtering, pagination, and hybrid (keyword + vector) search.

## What this is
- A front-end UI (HTML/CSS/JS) that:
  - Queries a Typesense collection (e.g., `bills_state`)
  - Supports keyword search + hybrid/vector search behavior
  - Provides filters for state, policy type, themes, entity type, status, and date ranges
  - Displays results as cards with summary text and outbound links (LegiScan + entity source)

## What this is not
- This repo does **not** include the backend ingestion pipeline that generates the dataset.
- This repo is **not** intended to be a general-purpose Typesense admin tool.
- This repo should not be used to host secrets: the API key used in the UI must be **search-only**.

---

## Live Links
- **Tracker:** https://us-state.techpolicytracker.com/
- **Home:** https://integrityinstitute.org/legislative-tracker

---

## Architecture (high-level)
- **Browser UI**
  - InstantSearch.js widgets for search box, hits, pagination, stats, refinement lists
  - Custom widgets:
    - State selector/filter (all 50 states + territories)
    - "Current legislative session only" checkbox refinement
    - Intro date range picker (Flatpickr + InstantSearch range connector)
- **Typesense**
  - Hosts the search collection and supports faceting + hybrid/vector search
- **Hybrid Search**
  - Uses a normalized query and applies vector/keyword weighting
  - When the query is empty: disables hybrid/vector and runs a normal faceted browse

---

## Files (typical)
- `index.html` – main page and layout
- `typesense-instantsearch-demo/src/app.js` – UI logic (InstantSearch configuration + widgets)
- `typesense-instantsearch-demo/src/app.css` – UI styling

> Note: some deployments may bundle JS/CSS differently. Adjust paths accordingly.

---

## Setup / Configuration

### 1) Typesense connection
In `app.js` (or equivalent), configure:

- `apiKey`: **search-only** key
- `nodes`: host + https port 443
- `indexName`: e.g. `bills_state`
- `query_by`: comma-separated field list (text fields only)

Example:

```js
const typesenseInstantsearchAdapter = new TypesenseInstantSearchAdapter({
  server: {
    apiKey: "SEARCH_ONLY_KEY",
    nodes: [{ host: "YOUR_HOST", port: "443", protocol: "https" }],
    connectionTimeoutSeconds: 10
  },
  additionalSearchParameters: {
    query_by: "Name,State,Introduced by,Themes,Bill Summary"
  }
});
```

### 2) Collection Schema
The Typesense collection should include these fields:

| Field | Type | Facet | Notes |
|-------|------|-------|-------|
| `id` | string | No | Unique identifier |
| `Name` | string | No | Bill number + title |
| `State` | string | Yes | State abbreviation |
| `Policy Type` | string | Yes | e.g., "Legislation" |
| `Themes` | string[] | Yes | Technology policy themes |
| `entity_type` | string | Yes | "House" or "Senate" |
| `Bill_status` | string | Yes | Current status |
| `date_of_introduction` | int64 | No | Unix timestamp |
| `Bill Summary` | string | No | AI-generated summary |
| `introduced_by` | string | No | Sponsor name |
| `legiscan_url` | string | No | Link to LegiScan |
| `state_url` | string | No | Link to state source |

### 3) Filters Configuration
State tracker includes additional filters compared to Federal:

- **State filter**: Multi-select for all 50 states + territories
- **Theme filter**: Technology policy categories
- **Entity type**: House vs Senate
- **Status**: Bill status (Introduced, Passed, etc.)
- **Date range**: Introduction date picker

---

## Differences from Federal Tracker

| Aspect | Federal | State |
|--------|---------|-------|
| Collection name | `bills_federal` | `bills_state` |
| State filter | Not needed (always "US") | Required (50+ options) |
| Data volume | Hundreds of bills | Thousands of bills |
| query_by fields | Excludes State | Includes State |

---

## Deployment
- Deploy static files to your hosting provider
- Ensure CORS is configured on Typesense if needed
- Use only **search-only** API keys in client-side code

---

## Related Repositories
- **Update Process:** [state-tracker-update-process](https://github.com/ramirza1/state-tracker-update-process) - Python scripts for data pipeline
- **Federal Tracker Site:** Similar structure for US Federal legislation
- **Federal Update Process:** [federal-tracker-update-process](https://github.com/ramirza1/federal-tracker-update-process)