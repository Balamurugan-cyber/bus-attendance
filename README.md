# Office Bus Attendance

Single-bus, single-route office attendance system. Node.js + Express API,
SQLite database, plain HTML/CSS/JS frontend (no build step).

Data model: one bus, one fixed route, so the only real entities are
**employees** and **attendance**. Each attendance row is uniquely keyed by
`(employee_id, attendance_date)`, with separate morning/evening status and
timestamps on that same row — this is what guarantees morning and evening
always belong to the same date.

## Run locally

```bash
npm install
npm start
```

Then open http://localhost:3000. A `data/attendance.db` SQLite file is
created automatically on first run, seeded with 3 sample employees.

## Project structure

```
server/
  index.js          Express app entry point
  db.js              SQLite connection + schema + seed data
  routes/
    employees.js     Employee CRUD
    attendance.js     Daily roll-call + mark present/absent
    reports.js        Monthly report + CSV export + date history
public/
  index.html, app.js, styles.css   Frontend (vanilla JS, talks to /api/*)
render.yaml          Render blueprint (see below)
```

## Deploying to Render

### Option A — Blueprint (one click, includes a persistent disk)

1. Push this folder to a new GitHub repository.
2. In the Render dashboard: **New → Blueprint**, connect the repo. Render
   reads `render.yaml` and creates the service for you.
3. Click **Apply**. First deploy takes a couple of minutes.

**Important:** `render.yaml` requests a persistent disk (`/data`) so the
SQLite file survives restarts and redeploys. **Persistent disks are not
available on Render's free plan** — if you deploy on the free tier, either:
- remove the `disk:` block from `render.yaml` and accept that the database
  resets on every redeploy/restart (fine for testing, not for real use), or
- upgrade the service to a paid plan that supports disks (a small one is
  enough for this app).

### Option B — Manual web service

1. Push the folder to GitHub.
2. Render dashboard → **New → Web Service** → connect the repo.
3. Settings:
   - **Build command:** `npm install`
   - **Start command:** `npm start`
   - **Environment variable:** `DB_PATH` = `/data/attendance.db` (only if
     you've attached a persistent disk at `/data` under the Disks tab —
     otherwise leave `DB_PATH` unset and it'll write to a local file that
     resets on redeploy).
4. Deploy.

## API reference

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/employees?status=active\|all` | List employees |
| POST | `/api/employees` | Add employee `{code, name, department?, phone?}` |
| PATCH | `/api/employees/:id` | Edit `{name?, department?, phone?}` |
| PATCH | `/api/employees/:id/status` | `{status: 'ACTIVE'\|'INACTIVE'}` |
| DELETE | `/api/employees/:id` | Permanently delete (cascades attendance) |
| GET | `/api/attendance?date=YYYY-MM-DD` | Daily roll call + summary |
| POST | `/api/attendance/mark` | `{employeeId, date, field: 'morning'\|'evening', status: 'PRESENT'\|'ABSENT'}` |
| GET | `/api/reports/monthly?month=YYYY-MM` | Per-employee monthly counts |
| GET | `/api/reports/monthly.csv?month=YYYY-MM` | Same, as CSV download |
| GET | `/api/reports/history?date=YYYY-MM-DD` | Raw history for a date |

## Notes on scaling this later

If the company later adds more buses or routes, the schema extends
cleanly: add `buses`/`routes` tables and a `trip_id` foreign key on
`attendance` instead of assuming a single fixed route. That's a bigger
version of this same app I can build if you need it.
