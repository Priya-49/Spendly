# Spec: Login and Logout

## Overview
This step wires up session-based authentication for Spendly. Users can sign in with their email and password, receive a Flask session cookie, and sign out to clear it. It is the gateway that unlocks all protected routes in later steps — without it, the dashboard, profile, and expense management pages cannot identify who is making a request.

## Depends on
- Step 01 — Database Setup (`get_db()` and the `users` table must exist)
- Step 02 — Registration (users must already exist in the database to log in)

## Routes
- `POST /login` — validate credentials, create session, redirect — public
- `GET /logout` — clear session, redirect to landing — public (no login required to log out)

The existing `GET /login` route (renders `login.html`) is already present in `app.py` and requires no changes beyond accepting the `error` template variable.

## Database changes
No database changes. Login reads from the existing `users` table; logout only touches the Flask session.

## Templates
- **Modify:** `templates/login.html` — already renders `{{ error }}` and posts to `POST /login`; no structural changes needed.
- **Modify:** `templates/base.html` — update the nav links block to show a "Sign out" link when the user is logged in (`session.get('user_id')` is truthy), and "Sign in" / "Get started" when they are not.

## Files to change
- `app.py` — implement `POST /login` and `GET /logout`; set `app.secret_key`; import `session`, `request`, `redirect`, `url_for` from Flask; import `check_password_hash` from `werkzeug.security`; import `get_db` from `database.db`
- `templates/base.html` — conditional nav links based on `session.get('user_id')`

## Files to create
No new files.

## New dependencies
No new dependencies. `werkzeug` is already installed as a Flask dependency.

## Rules for implementation
- No SQLAlchemy or ORMs — use raw `sqlite3` via `get_db()`
- Parameterised queries only — never interpolate values into SQL strings
- Passwords verified with `werkzeug.security.check_password_hash` — never compare plaintext
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- `app.secret_key` must be set before any session use; use `"dev-secret-change-in-prod"` for now
- Store only `user_id` and `user_name` in the session — never store the password hash
- On failed login (wrong password or unknown email), re-render `login.html` with `error="Invalid email or password."` — do not redirect, do not distinguish between the two failure cases
- On successful login, redirect to `url_for('landing')` until a dashboard route exists
- `GET /logout` must call `session.clear()` then redirect to `url_for('landing')`
- Do not use `flask_login` or any auth extension

## Definition of done
- [ ] Submitting the login form with a valid email and correct password creates a session and redirects away from `/login`
- [ ] Submitting with an incorrect password re-renders `login.html` with a visible error and no redirect
- [ ] Submitting with an email that does not exist re-renders `login.html` with a visible error
- [ ] After login the nav bar shows a "Sign out" link instead of "Sign in" / "Get started"
- [ ] Visiting `/logout` clears the session and redirects to the landing page
- [ ] After logout the nav bar reverts to showing "Sign in" / "Get started"
- [ ] Refreshing any page after login still shows the logged-in nav state (session persists across requests)
