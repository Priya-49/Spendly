# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the app

```bash
# Activate the virtual environment (Windows)
venv\Scripts\activate

# Start the development server
python app.py
```

App runs at **http://localhost:5001** with Flask debug mode on (auto-reloads on file changes).

## Running tests

```bash
pytest
# Single file
pytest tests/test_routes.py
```

## Architecture

**`app.py`** is the single entry point — all Flask routes live here. There is no blueprint structure. Routes are grouped into implemented ones and placeholder stubs (marked with step numbers) that students fill in progressively.

**Template inheritance** — every page extends `base.html`, which provides the navbar, footer (with Terms/Privacy links), and loads `static/css/style.css` and `static/js/main.js`. Child templates override `{% block title %}`, `{% block content %}`, and optionally `{% block head %}` (for page-specific CSS) and `{% block scripts %}`.

**`database/db.py`** is a stub for a future SQLite layer. It will expose `get_db()` (connection with row factory + foreign keys enabled), `init_db()` (CREATE TABLE IF NOT EXISTS), and `seed_db()` (dev sample data). The DB file `expense_tracker.db` is gitignored.

**CSS design system** — all design tokens (colors, fonts, radii, spacing) are CSS custom properties in the `:root` block at the top of `style.css`. Use these variables (`--ink`, `--accent`, `--paper`, `--border`, etc.) for any new styles rather than hardcoding values. New page-scoped styles can be added at the bottom of `style.css` above the responsive section.

## Key conventions

- Navigation links in `base.html` use `url_for()` — add a named route in `app.py` before adding a link.
- Placeholder routes return a plain string (e.g. `"Logout — coming in Step 3"`); replace the body with `render_template(...)` when implementing.
- The SQLite DB path and any secrets should go in a `.env` file (gitignored); load with `python-dotenv` when needed.
