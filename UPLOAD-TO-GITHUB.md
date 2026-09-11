# How to put this on GitHub and get it online

No command line needed. Three steps, about five minutes.

## Step 1 — Upload the code to GitHub

1. Go to <https://github.com/jayaraman-sathish/montra-fleet-maintenance>
2. Click **Add file** → **Upload files**
3. Unzip the file you were sent, then drag **everything inside the
   `montra-fleet-maintenance` folder** into the browser window
   (the `db`, `src`, `web` and `docs` folders, and the loose files like
   `README.md` and `render.yaml`)
4. Scroll down, click **Commit changes**

Wait for the upload to finish — it is a few hundred small files.

## Step 2 — Create the hosting on Render

1. Go to <https://dashboard.render.com>
2. Click **New** → **Blueprint**
3. Choose the `montra-fleet-maintenance` repository
4. Render reads `render.yaml` and offers to create two things:
   a PostgreSQL database and a web service. Click **Apply**.

Render builds and starts the application, creates the database, loads the
schema and the demonstration fleet. First deploy takes roughly five minutes.

## Step 3 — Open it

Render gives you a web address ending in `.onrender.com`.

Sign in with any of these — the password for all of them is `montra123`:

| Username | What they see |
|---|---|
| `ravi.kumar` | Service Centre Manager — the Chennai workshop |
| `kumar.s` | Technician — assigned jobs and the checklist |
| `priya.n` | QC Supervisor — the release gate |
| `vijay.r` | Service Coordinator — breakdowns and triage |
| `fleet.abc` | Fleet Customer — only ABC Logistics vehicles |
| `admin` | Administrator — configuration and audit |

Signing in as two different people is the fastest way to show Montra that
access control is real: the Chennai manager sees 28 vehicles, the fleet
customer sees only their own 20, out of 82 in the system.

---

### A note on the free Render plan

Free services sleep after 15 minutes of no traffic and take about 50 seconds
to wake on the next visit. Fine for demonstrations. Before showing it to
Montra, open it once a few minutes beforehand so it is already awake.

The free database is removed after 30 days. For anything beyond a
demonstration, move both to a paid plan.

### Turning off the demo data

The demonstration fleet is loaded because `SEED_DEMO_DATA` is set to `true`
in `render.yaml`. For a real deployment, change that to `false` before the
first deploy.
