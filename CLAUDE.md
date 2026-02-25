# CLAUDE.md — AI Assistant Guide for resume-search

## Project Overview

Algolia-powered resume search application for VietnamWorks (Vietnamese job portal). Provides instant full-text search with faceted filtering over a resume dataset synced from MySQL to an Algolia index.

**Two main parts:**
1. **Frontend** — Client-side search UI (HTML/JS/CSS) that queries Algolia directly
2. **Backend** — PHP script that syncs resume data from MySQL into the Algolia index

## Repository Structure

```
├── index.html                 # Main search page (instant search UI)
├── autocomplete.html          # Alternative autocomplete demo page
├── search.css                 # Large compiled/vendor CSS (542KB, do not edit)
├── conf/
│   ├── config.js              # Frontend Algolia credentials (app ID, API key, index)
│   └── config.php             # Backend DB + Algolia credentials
├── js/
│   ├── app.js                 # Main application logic (search, facets, rendering)
│   ├── moment.js              # Date/time formatting library
│   └── accounting.min.js      # Number formatting library
├── css/
│   └── styles.css             # Application-specific styles
├── src/
│   ├── index.php              # Primary data sync script (MySQL → Algolia)
│   └── index_with_fields_array.php  # Alternative sync variant
├── img/                       # Static image assets
├── package.json               # Node.js deps (BrowserSync only)
├── composer.json               # PHP deps (Algolia PHP client)
└── .gitignore
```

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Search engine | Algolia (cloud SaaS) |
| Frontend | jQuery 2.1.4, Bootstrap 3.3.6, Hogan.js 3.0.2, Lodash 4.6.1 |
| Backend | PHP 5.x+, MySQL |
| Dev server | BrowserSync 2.8.1 |
| Libraries loaded via CDN | algoliasearch 3.x, algoliasearch-helper 2.x, Ion Range Slider 2.0.12, Font Awesome 4.5 |

## Development Commands

```bash
# Install frontend deps
npm install

# Install PHP deps
composer install

# Start dev server with live reload (watches css/*, img/*, js/*, index.html)
npm run dev
# → opens BrowserSync at http://localhost:3000

# Sync resumes from MySQL to Algolia (run from src/)
php src/index.php
```

## Key Files to Know

- **`js/app.js`** — Core frontend logic (~464 lines). Initializes Algolia search helper, binds search input, renders facets (location, category, job level, salary, experience, language, nationality, employer), handles pagination, manages URL state for bookmarkable searches. Uses Hogan.js templates defined inline.
- **`src/index.php`** — Backend sync script (~415 lines). Connects to MySQL, batch-processes resumes (1000/batch), extracts bilingual fields (Vietnamese/English), calculates credits, and pushes to Algolia index. Supports incremental delta sync via timestamps.
- **`css/styles.css`** — App-specific styles (~504 lines). Two-column layout: 23% left sidebar for facets, 77% right panel for results.
- **`conf/config.js`** — Frontend config: `APPLICATION_ID`, `SEARCH_ONLY_API_KEY`, `INDEX_NAME`, `RESUME_DETAIL_LINK_PREFIX`.
- **`conf/config.php`** — Backend config: Algolia credentials, MySQL connection, batch sizes.

## Architecture & Data Flow

1. **MySQL** holds source resume data
2. **`src/index.php`** extracts, transforms (bilingual fields, salary, credits), and loads into Algolia
3. **Algolia** indexes and serves search queries
4. **`index.html` + `js/app.js`** queries Algolia client-side, renders results with Hogan.js templates

All search queries go directly from the browser to Algolia's API — there is no application server in the request path for search.

## Conventions & Patterns

- **No build step** for frontend — plain JS files loaded via `<script>` tags and CDN links. No transpilation, bundling, or minification pipeline.
- **Hogan.js templates** are defined as inline `<script type="text/template">` blocks in `index.html` and rendered in `app.js`.
- **Bilingual data** — Many fields (location, category, job level, experience, nationality) store both Vietnamese and English values, extracted from delimited strings in the database.
- **Facets** are rendered client-side from Algolia helper responses. Each facet type has its own render function in `app.js`.
- **Range sliders** (salary, date) use Ion Range Slider with custom formatting via moment.js and accounting.js.
- **URL state** is managed manually via query parameters for bookmarkable searches.

## Sensitive Files — Do Not Commit

These files contain secrets and are gitignored:
- `src/config.php` — database credentials
- `conf/config.js` — should have empty credential values in the repo
- `vendor/` — Composer dependencies
- `node_modules/` — npm dependencies

## Testing & CI

- **No automated tests** are configured
- **No CI/CD pipelines** exist
- **No linting** tools are set up
- Manual testing via BrowserSync dev server

## Common Tasks

### Modifying search behavior or adding facets
Edit `js/app.js`. Search-related configuration (facets, disjunctive facets, numeric refinements) is set up in the `algoliasearchHelper` initialization near the top of the file.

### Changing result card appearance
Edit the Hogan.js template in `index.html` (look for `<script type="text/template">` tags) and corresponding styles in `css/styles.css`.

### Updating the data sync
Edit `src/index.php`. The SQL query and field mappings determine what data reaches the Algolia index.

### Adding new configuration
Frontend config goes in `conf/config.js`, backend config in `conf/config.php`.
