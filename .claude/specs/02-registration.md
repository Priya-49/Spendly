# Spec: Registration

## Overview
Implements user account creation for Spendly. A visitor fills in their name,
email, and password on the `/register` page. The app validates the input,
hashes the password, inserts the new user into the `users` table, and
redirects them to the login page with a success flash message. Duplicate
email addresses are rejected with a clear inline error. This is the entry
point for all authenticated features that follow in the roadmap.

## Depends on
- Step 1 — Database Setup (`database/db.py` must expose `get_db()` and the
  `users` table must exist)

## Routes
- `GET  /register` — render the registration form — public
- `POST /register` — process form submission, create account, redirect — public

## Database changes
No new tables or columns. Uses the existing `users` table:
```
id            INTEGER PRIMARY KEY AUTOINCREMENT
name          TEXT    NOT NULL
email         TEXT    UNIQUE NOT NULL
password_hash TEXT    NOT NULL
created_at    TEXT    DEFAULT (datetime('now'))
```

## Templates
- **Modify:** `templates/register.html`
  - Must extend `base.html`
  - Form with fields: `name`, `email`, `password`, `confirm_password`
  - Display flash messages (errors and success)
  - Client-side: mark fields invalid if flash errors are present
  - Link to `/login` for users who already have an account

## Files to change
- `app.py` — convert `GET /register` stub into a proper GET+POST view;
  add `POST /register` handler; import `get_db`, `generate_password_hash`,
  `flash`, `redirect`, `url_for`, `request`
- `templates/register.html` — replace placeholder with functional form

## Files to create
- `database/db.py` — implement `get_db()`, `init_db()`, `seed_db()` if not
  already done by Step 1 (this spec assumes Step 1 is complete before
  implementation begins)

## New dependencies
No new dependencies. `werkzeug` is already installed as a Flask dependency.

## Rules for implementation
- No SQLAlchemy or ORMs — use `sqlite3` directly via `get_db()`
- Parameterised queries only — never interpolate user input into SQL strings
- Passwords hashed with `werkzeug.security.generate_password_hash`; never
  store plain-text passwords
- Use CSS variables — never hardcode hex values in templates or stylesheets
- All templates extend `base.html`
- Use Flask `flash()` for user-facing messages; never render raw error strings
- Validate server-side: name non-empty, valid email format, password >= 8
  characters, passwords match, email not already registered
- Redirect to `GET /login` after successful registration (Post/Redirect/Get
  pattern)

## Definition of done
- [ ] `GET /register` renders the form with no errors
- [ ] Submitting valid details creates a new row in `users` and redirects to
      `/login` with a "Account created — please log in" flash message
- [ ] Submitting a duplicate email shows an inline error and does not insert a
      new row
- [ ] Submitting mismatched passwords shows an inline error
- [ ] Submitting a password shorter than 8 characters shows an inline error
- [ ] Submitting an empty name shows an inline error
- [ ] Password is stored as a hash — never plain text — verifiable by
      inspecting the `users` table directly
- [ ] The form re-populates `name` and `email` (but not password) after a
      failed submission
- [ ] The "Already have an account? Log in" link navigates to `/login`
