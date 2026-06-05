# Spec: Registration

## Overview
This step implements user account creation for Spendly. A visitor fills in their name, email, and password on `/register`; the server validates the input, hashes the password, and inserts a new row into the `users` table. On success the user is redirected to `/login`. On failure the form re-renders with a clear error message. The registration template already exists — this step makes it functional.

## Depends on
- Step 01 — Database Setup (`get_db()` and the `users` table must exist)

## Routes
- `POST /register` — validate input, hash password, insert user, redirect to `/login` — public

The existing `GET /register` route (renders `register.html`) is already present in `app.py` and requires no changes.

## Database changes
No database changes. Registration writes to the existing `users` table.

## Templates
- **Modify:** `templates/register.html` — already renders `{{ error }}` and posts to `POST /register`; no structural changes needed. Fields `name`, `email`, `password` are already in the form.

## Files to change
- `app.py` — implement `POST /register`; import `request`, `redirect`, `url_for` from Flask (if not already); import `generate_password_hash` from `werkzeug.security`; import `get_db` from `database.db`

## Files to create
No new files.

## New dependencies
No new dependencies. `werkzeug` is already installed.

## Rules for implementation
- No SQLAlchemy or ORMs — use raw `sqlite3` via `get_db()`
- Parameterised queries only — never interpolate values into SQL strings
- Passwords hashed with `werkzeug.security.generate_password_hash` — never store plaintext
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- Validate server-side (not just HTML `required`): name, email, and password must all be non-empty strings
- Minimum password length: 8 characters — re-render the form with an error if shorter
- Duplicate email: catch `sqlite3.IntegrityError` and re-render the form with the message "An account with that email already exists."
- On success: redirect to `url_for('login')` — do not log the user in automatically
- Do not use `flask_login` or any auth extension

## Definition of done
- [ ] Submitting the form with valid name, email, and password creates a new row in `users` and redirects to `/login`
- [ ] The stored `password_hash` is a werkzeug hash string, not the original plaintext
- [ ] Submitting with a missing name, email, or password re-renders `/register` with a visible error and no redirect
- [ ] Submitting with a password shorter than 8 characters re-renders `/register` with a visible error
- [ ] Submitting with an email that already exists re-renders `/register` with "An account with that email already exists."
- [ ] Submitting twice with the same email does not create a duplicate row
