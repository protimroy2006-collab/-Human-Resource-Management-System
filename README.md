# Enigma's Management

> A locally-hosted, dark-mode HR command deck with role-based access, combination-code security, and a leave-routing workflow.

## Architecture

- **Backend:** FastAPI (Python) + Motor (async MongoDB)
- **Frontend:** React 19 + Tailwind + Framer Motion + Phosphor icons
- **Database:** MongoDB (single node, local)
- **Auth:** JWT (httpOnly cookies) + bcrypt + 12-char combination code
- **Everything is local.** No external APIs, no third-party auth, no cloud storage.

---

## Feature list

- Dynamic dark-mode landing page with an asymmetric bento grid.
- Three strictly separated dashboards: **Admin**, **Employee**, **Subordinate**.
- **Combination-based security**: login requires `email + password + XXXX-XXXX-XXXX combination code`.
- **Character-limited display security**: combo codes and emails are masked when shown in audit views.
- **Hardcoded admin cap** (`MAX_ADMINS=5`) — the sixth admin is refused at the API layer.
- **Admin bootstrap code** — a shared secret required to seat any additional admin.
- **Leave application** flow from Subordinate → Admin queue with approve/reject and optional decision notes.
- **Navigation audit** — every route change is logged server-side with the user, IP, and prev-path.
- **Skeleton loaders** everywhere (`.skeleton` shimmer class + bento-shaped placeholders).
- **Rate limiting**: 30 req/min per IP on `/api/auth/*`; 5 failed logins → 15-min lockout (HTTP 423).
- **Persistent sessions** via httpOnly `enigma_access` (8h) + `enigma_refresh` (7d) cookies.

---

## File structure

```
enigma-management/
├── backend/
│   ├── server.py                # FastAPI entrypoint, startup seed, CORS, indexes
│   ├── security.py              # bcrypt hashing, JWT, combo helpers, masking
│   ├── deps.py                  # Rate-limit, brute-force, audit helpers
│   ├── models.py                # Pydantic schemas + document models
│   ├── routes/
│   │   ├── auth.py              # /api/auth/*  (register, login, logout, me, refresh)
│   │   ├── leaves.py            # /api/leaves/* (create, mine, pending, decide, cancel)
│   │   └── users.py             # /api/users, /api/system/status, /api/audit*
│   ├── requirements.txt
│   └── .env
└── frontend/
    ├── src/
    │   ├── App.js               # Routes + auth provider + nav audit
    │   ├── index.css            # Design tokens, bento, skeleton shimmer, animations
    │   ├── context/AuthContext.jsx
    │   ├── lib/api.js           # Axios instance (withCredentials)
    │   ├── components/
    │   │   ├── Header.jsx
    │   │   ├── Footer.jsx
    │   │   ├── DashboardShell.jsx
    │   │   ├── NavAudit.jsx
    │   │   ├── ProtectedRoute.jsx
    │   │   ├── skeletons/PageSkeleton.jsx
    │   │   └── leaves/{LeaveForm.jsx,LeaveRow.jsx}
    │   ├── pages/
    │   │   ├── Landing.jsx
    │   │   ├── Login.jsx
    │   │   ├── Signup.jsx
    │   │   ├── NotFound.jsx
    │   │   └── dashboards/{AdminDashboard,EmployeeDashboard,SubordinateDashboard}.jsx
    │   └── components/ui/       # Shadcn primitives
    ├── package.json
    └── .env
```

---

## Database setup

Enigma uses a single MongoDB database (`DB_NAME`, defaults to `test_database`). All collections are created lazily. Indexes are provisioned automatically on backend startup:

| Collection          | Indexes                                                        |
| ------------------- | -------------------------------------------------------------- |
| `users`             | `email` (unique), `id` (unique)                                |
| `leaves`            | `id` (unique), `subordinate_id`, `status`                      |
| `audit_events`      | `created_at`                                                   |
| `login_attempts`    | `identifier` (unique)                                          |

To run MongoDB locally:

```bash
# macOS
brew install mongodb-community && brew services start mongodb-community

# Ubuntu
sudo apt install -y mongodb
sudo systemctl start mongod
```

---

## Environment variables

`backend/.env`

```
MONGO_URL="mongodb://localhost:27017"
DB_NAME="enigma_management"
CORS_ORIGINS="http://localhost:3000"
JWT_SECRET="<paste a random 64-char hex string here>"
ADMIN_EMAIL="admin@enigma.local"
ADMIN_PASSWORD="Enigma@Admin1"
ADMIN_COMBO="AX7K-9M2P-QR84"
MAX_ADMINS="5"
```

`frontend/.env`

```
REACT_APP_BACKEND_URL=http://localhost:8001
```

---

## Run locally

```bash
# 1. Backend
cd backend
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
uvicorn server:app --host 0.0.0.0 --port 8001 --reload

# 2. Frontend (new shell)
cd frontend
yarn install
yarn start
```

Open `http://localhost:3000`. The root admin is auto-seeded from the `.env` values on startup.

---

## Account creation

There are three ways to get in:

**A. First run.** If the database has zero admins, the signup page will show a *First run* banner and let anyone create the founding **Admin** without any code. After that, this route is closed.

**B. Invite code (recommended).** An existing admin generates a single-use code from `Admin → Invites`, sets the role (Admin / Employee / Subordinate) and an optional expiry, and shares the copied link (`/signup?code=XXXX-XXXX-XXXX`). The signup page verifies the code, locks the role, and consumes it on registration.

**C. Open signup for non-admin roles.** Without a code, anyone can still self-register as an **Employee** or **Subordinate**. **Admin** always requires either an invite code or the legacy `ADMIN_COMBO` shared code from `.env`.

All roles require:
- A **12-char combination** in the format `XXXX-XXXX-XXXX` (letters/digits, case-insensitive) that you choose yourself.
- A password of at least 8 characters.

**Root admin credentials (seeded from `.env`)**
- Email: `admin@enigma.local`
- Password: `Enigma@Admin1`
- Combo: `AX7K-9M2P-QR84`

The seeded admin exists so you can always sign in for the very first time. Once you are in, generate invite codes from `Admin → Invites` for everyone else.

---

## Login / logout / session persistence

**Login** — `/login` requires ALL THREE of: email, password, and combination code. Any mismatch counts as a failed attempt. After 5 failures against the same IP+email pair, the account is locked for 15 minutes (HTTP 423).

**Session persistence** — On a successful login/register the backend sets two httpOnly cookies:

- `enigma_access` (8 hours) — JWT used to authenticate each request.
- `enigma_refresh` (7 days) — used by `POST /api/auth/refresh` to mint a new access token when the old one expires.

The frontend calls `GET /api/auth/me` on every page load; if the cookie is valid you are silently restored to your dashboard. This means:
- Refreshing the page keeps you signed in.
- Closing and re-opening the browser keeps you signed in (up to the refresh cookie's 7-day life).
- Servers restart safely — no in-memory state.

**Logout** — `/api/auth/logout` clears both cookies. The frontend also drops its React context state and redirects to `/`.

---

## Leave workflow

1. A **Subordinate** (or **Employee**) opens their dashboard and files a leave request from `Apply leave` (or the Employee → `Leaves` tab).
2. The request lands in `POST /api/leaves` and appears under **Admin → Leave queue** with status `pending`.
3. An admin approves or rejects with an optional note. The change is written to `leaves` and emitted to `audit_events`.
4. The requester sees the updated status on their `History` tab. They can also cancel a request while it is still pending.

Non-admin roles cannot decide leave. Non-admins cannot view other users' leaves.

---

## Auditing

- Every route change fires `POST /api/audit/nav` with `path` + `from_path`.
- Every auth event and leave decision writes an audit entry.
- **Admin → Audit ledger** page renders the last 200 events with masked emails (character-limited display security).

---

## Security controls at a glance

| Control                           | Where                                                     |
| --------------------------------- | --------------------------------------------------------- |
| bcrypt password hashing           | `backend/security.py::hash_secret`                        |
| bcrypt combo-code hashing         | Same file — the combo is never stored in plaintext        |
| JWT with two-token model          | `security.py` + `routes/auth.py::_set_auth_cookies`       |
| httpOnly, `SameSite=None; Secure` | `routes/auth.py::_cookie_kwargs`                          |
| Rate limit (30/min per IP)        | `deps.py::rate_limit`                                     |
| Failed-login brute-force lock     | `deps.py::register_failed_login` + `check_lockout`        |
| Admin cap (5)                     | `routes/auth.py::register`                                |
| Admin bootstrap code              | `ADMIN_COMBO` env — required for `role="admin"` signup    |
| Nav audit                         | `routes/users.py::record_nav` + `NavAudit` component      |
| Character-limited email display   | `security.py::mask_email` used in audit + manager picker  |

---

## API reference (all under `/api`)

**Auth**
- `POST /auth/register` `{name,email,password,combo,role,invite_code?,admin_code?,manager_id?}`
- `POST /auth/login` `{email,password,combo}`
- `POST /auth/logout` (auth)
- `GET /auth/me` (auth)
- `POST /auth/refresh`

**Invites** (single-use signup codes)
- `POST /invites` (admin) `{role,note?,expires_in_days?}`
- `GET /invites` (admin) — list all
- `POST /invites/{id}/revoke` (admin)
- `GET /invites/verify?code=XXXX-XXXX-XXXX` (public)

**Leaves**
- `POST /leaves` (subordinate/employee)
- `GET /leaves/mine` (auth)
- `GET /leaves/pending` (admin)
- `GET /leaves/all` (admin/employee)
- `POST /leaves/{id}/decision` (admin) `{decision:"approved"|"rejected", note?}`
- `POST /leaves/{id}/cancel` (auth, owner only)

**Users / Audit / System**
- `GET /users` (admin/employee)
- `GET /users/managers` (public; masked emails)
- `PATCH /users/{id}/active?active=true|false` (admin)
- `GET /system/status` (admin)
- `GET /audit?limit=` (admin)
- `GET /audit/mine` (auth)
- `POST /audit/nav` `{path,from_path?}` (auth)

---

## Git

The repo is initialised as a standard Git project. Recommended `.gitignore` already excludes `.env` files, `node_modules`, and Python virtualenvs. Commit your `.env.example` (not `.env`) for teammates.

## License / status

MIT-licensed. Internal-tool grade.
