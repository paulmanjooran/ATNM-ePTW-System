# ATNM Digital Permit to Work (ePTW)

Al Tasnim Group — Non-PDO Operations. A Control-of-Work platform for planning,
authorizing, controlling and assuring permit-to-work activity across all
divisions, projects and sites.

**Stack: Google Sheets (database) + Google Apps Script (API) + a single static
HTML file (frontend, hosted on GitHub Pages). No other cloud service.**

```
GitHub Pages (index.html)
        │  fetch() → POST → Apps Script /exec URL
        ▼
Apps Script Web App (Code.gs, bound to the Sheet)
        │  reads/writes
        ▼
Google Sheet (the database)
```

---

## Repo contents

| File | Goes where | Purpose |
|---|---|---|
| `index.html` | GitHub Pages | The entire frontend — every screen, all styling, all client logic. One file, no build step. |
| `Code.gs` | Google Apps Script (inside the Sheet) | The entire backend — API routing, authentication, email, all data reads/writes. |

These two files never share a repo or a folder. `Code.gs` lives inside the
Google Sheet's script editor; it is **not** uploaded to GitHub.

---

## One-time setup

### 1. Google Sheet + Apps Script (backend)

1. Create a new blank Google Sheet — name it something like "ATNM ePTW Database".
2. **Extensions → Apps Script**. Delete any starter code, paste in the entire
   contents of `Code.gs`.
3. At the top of `Code.gs`, confirm `REQUIRED_EMAIL_DOMAIN` matches your real
   corporate domain (ships set to `@altasnim.com`).
4. In `setupSheet()`, replace every placeholder email address with real
   `@altasnim.com` addresses for each seeded user — this is what temporary
   passwords, password-reset, and notification emails are sent to.
5. Run the **`setupSheet`** function once (function picker at the top of the
   editor → select it → ▶ Run). Approve the permission prompts — this script
   only touches this one spreadsheet plus your own Gmail for sending mail; it
   requests no other scope.
   - Creates all 8 tabs with headers: `Config`, `Users`, `Employees`,
     `Permits`, `Actions`, `AuditLog`, `Auth`, `Sessions`.
   - Seeds demo users, employees, and default config/scoring weights.
   - Generates a **unique password per user**, emails it to them, and — for
     anyone without a valid email — lists it instead in a temporary
     `InitialCredentials` tab (distribute those manually, then delete the tab).
6. **Deploy → New deployment → type: Web app.**
   - Execute as: **Me**
   - Who has access: **Anyone** (not "Anyone within [domain]" — that blocks
     the GitHub-hosted frontend from ever reaching it; domain restriction is
     already enforced inside the login logic instead)
7. Click **Deploy**, authorize again if asked, and copy the URL ending in
   `/exec`. That's your backend endpoint.

Re-deploying after any code edit: **Deploy → Manage deployments → pencil icon
→ New version → Deploy** — this keeps the same `/exec` URL and just updates
what's behind it.

### 2. GitHub Pages (frontend)

1. New repository → upload `index.html` to the root.
2. **Settings → Pages** → Source: **Deploy from a branch** → Branch: `main`,
   folder `/ (root)` → Save.
3. Live in a minute or two at `https://<you>.github.io/<repo>/`.

### 3. Connect them

1. Open your GitHub Pages URL.
2. Login screen → **"Connect Google Sheet"** link (or once inside, Admin
   Console → Integration) → paste the `/exec` URL → Save.
3. Sign out and back in with a real Employee ID and the password from your
   email, to verify the connection.

---

## Signing in

- **Username** = Employee ID (unique per person, e.g. `ENG-2201`).
- **Password** = whatever was emailed at account creation, or set via
  Change Password / Forgot Password.
- Only accounts with a registered `@altasnim.com` email can sign in — checked
  server-side on every login attempt.
- **No backend connected yet?** The app still runs in a local demo mode with
  in-browser sample data — any seeded Employee ID + password `Welcome@1`.
  This mode is clearly labelled as insecure/demo-only in the UI; nothing
  persists between sessions and no real authentication happens client-side.

---

## What's implemented

Full permit lifecycle: field verification gate → application → digital
approval → HSE concurrence (risk-based) → issue → holder acceptance → AI
trilingual (EN/AR/HI) toolbox talk with voice playback → start work check →
active monitoring with live countdown → daily applicant field visits → HSE
field assurance reporting → suspension/revalidation → handback → closure.

Also: role-based dashboards, a standalone HSE Dashboard with filters and
charts, live performance-alert feed (compliance thresholds, repeat findings,
missed visits, expired-while-active, etc.), permit register with search/
filter, SIMOPS conflict banner, corrective actions register, full audit
trail, and an Admin Console covering integration, users (add/edit/
deactivate/remove), permit types, risk matrix, and configurable scoring
weights.

Authentication: unique per-user hashed+salted passwords (SHA-256, salted,
stored and updated in place — never in plaintext), session tokens, Change
Password, Forgot Password (emailed temporary password), domain-restricted
sign-in, and email notifications on key events (submission, approval,
issue, suspension, stop-work, corrective actions, password changes).

## Known limitations — read before a real rollout

- **Email domain validation is format-only.** The backend checks the email
  *string* ends in `@altasnim.com`; it does not verify via Google that the
  person actually owns that address. True identity-verified sign-in would
  mean routing through Google Workspace SSO/OAuth — a separate, larger piece
  of work, not what's built here.
- **SIMOPS detection in the demo data is illustrative, not live.** The one
  conflict you'll see is a pre-linked demo pair; creating two new overlapping
  permits today will not currently trigger an automatic alert. A real overlap
  engine hasn't been built yet.
- **MailApp quota**: roughly 100 emails/day on a plain Gmail account (higher
  on Google Workspace). Fine for a pilot team; revisit if usage scales.
- **Apps Script Web App limits**: ~6 minute execution timeout per request,
  modest concurrent-request quota. Fine for a pilot; a real production
  rollout at scale would need a proper backend/database eventually.
- Some admin screens (permit types, risk/approval matrix, hazard/control
  library) are still view-only, not yet editable from the UI.
- Performance Alerts and email notifications cover the highest-value trigger
  events from the original spec, not literally every one listed.

## Support / next steps

This is a working pilot-grade prototype, not a hardened enterprise system.
Treat the limitations above as a punch list, not a hidden risk — they're
documented here specifically so nothing is discovered the hard way during a
live demo.
