# Spec: Database Setup

## Overview
This step establishes the data layer that every other feature depends on. It implements `database/db.py` — a single module that provides a `get_db()` connection factory, an `init_db()` function that creates all tables, and a `seed_db()` function for development data. The SQLite database file (`expense_tracker.db`) already exists at the project root; this step wires up the Python interface to it.

## Depends on
Nothing. This is the foundation step.

## Routes
No new routes. `init_db()` will be called once from the Flask app startup or a CLI command, not via HTTP.

## Database changes
Create the following tables (using `CREATE TABLE IF NOT EXISTS`):

**users**
```
id            INTEGER PRIMARY KEY AUTOINCREMENT
name          TEXT    NOT NULL
email         TEXT    UNIQUE NOT NULL
password_hash TEXT    NOT NULL
created_at    TEXT    DEFAULT (datetime('now'))
```

**expenses**
```
id          INTEGER PRIMARY KEY AUTOINCREMENT
user_id     INTEGER NOT NULL REFERENCES users(id)
amount      REAL    NOT NULL
category    TEXT    NOT NULL
date        TEXT    NOT NULL
description TEXT
created_at  TEXT    DEFAULT (datetime('now'))
```

Both tables already exist in `expense_tracker.db` — `CREATE TABLE IF NOT EXISTS` ensures `init_db()` is idempotent.

## Templates
No template changes.

## Files to change
- `database/db.py` — implement `get_db()`, `init_db()`, `seed_db()`
- `app.py` — call `init_db()` on startup using the Flask app context (`with app.app_context()`)

## Files to create
No new files.

## New dependencies
No new dependencies. `sqlite3` is part of the Python standard library.

## Rules for implementation
- No SQLAlchemy or ORMs — use raw `sqlite3` only
- Parameterised queries only — no string formatting in SQL
- Passwords hashed with `werkzeug.security.generate_password_hash` in `seed_db()`
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- `get_db()` must set `conn.row_factory = sqlite3.Row` so columns are accessible by name
- `get_db()` must enable foreign keys with `PRAGMA foreign_keys = ON`
- `get_db()` must use the path `expense_tracker.db` relative to the project root — derive it with `os.path.join(os.path.dirname(__file__), '..', 'expense_tracker.db')`
- `init_db()` must use `CREATE TABLE IF NOT EXISTS` — never `DROP TABLE`
- `seed_db()` should insert at least one sample user with a hashed password

## Definition of done
- [ ] `from database.db import get_db, init_db` works without error in `app.py`
- [ ] Running `init_db()` from a Flask shell creates both tables (or does nothing if they already exist)
- [ ] `get_db().execute("SELECT * FROM users").fetchall()` returns rows as `sqlite3.Row` objects accessible by column name
- [ ] Foreign key enforcement is active: inserting an expense with a non-existent `user_id` raises an `IntegrityError`
- [ ] `seed_db()` inserts a sample user; the stored password is a hash, not plaintext
