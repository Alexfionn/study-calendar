# Study Calendar → Notion

A mobile-friendly task entry interface for your Notion `Study Calendar` database.
Add new study tasks from your phone with a calendar picker, module multi-select, and status slider.

## Architecture

```
Phone (HTML on GitHub Pages)
   │
   │   read modules.json
   │◄──────────────────────────┐
   │                           │
   │   POST repository_dispatch│
   ▼                           │
GitHub Actions Workflows       │
   │   (1) study_task          │  (2) nightly cron
   ▼                           │   syncs Module options
Notion Database ───────────────┘
```

Two workflows:
- **`notion.yml`** — runs when you submit a task, creates the Notion page
- **`sync-modules.yml`** — runs nightly (and on demand), refreshes `modules.json`

---

## Prerequisites — your existing Notion DB

You said you already have a `Study Calendar` database with these properties:

| Property  | Type         |
| --------- | ------------ |
| `Name`    | Title        |
| `Date`    | Date         |
| `Module`  | Multi-select |
| `Status`  | Status       |

The `Status` property must contain at least these options:
- `Not started`
- `In progress`
- `Done`
- ` ` (a single space — for items excluded from automations)

> ⚠ Property names must match **exactly** (case + spaces).

---

## Setup — one time, ~15 minutes

### 1 · Reuse Notion integration (or create a new one)

You already have an integration from the daily tracker (`Daily Track` or whatever you named it). You can:

**Option A:** reuse it — share the `Study Calendar` DB with that same integration.
**Option B:** create a new one for clean separation (`Study Cal`).

To share:
1. Open the `Study Calendar` DB in Notion
2. Top right: **„…"** → **Connections** → **Connect to** → pick your integration → **Confirm**

### 2 · Database-ID copy

Open the DB as a full page. The URL looks like:
```
https://www.notion.so/your-workspace/abc123def456...?v=xyz...
                                      └─── DB-ID ───┘
```
The **32-char block** between `notion.so/...` and `?` is your Database ID.

> ⚠ Don't confuse this with the View ID that comes after `?v=`. You want the part **before** the `?`.

### 3 · Create the GitHub repo

1. Go to https://github.com/new
2. Name: `study-calendar`
3. **Public** (Pages requires public on free accounts)
4. ☑ **Add a README file**
5. **Create repository**

### 4 · Upload the files

Upload these to the new repo:
- `index.html` (root)
- `modules.json` (root, the empty placeholder)
- `.gitignore` (root)

For the workflows — they go into a sub-folder:
1. **Add file** → **Create new file**
2. File name: `.github/workflows/notion.yml`
3. Paste contents from the provided `notion.yml`
4. Commit
5. Repeat for `.github/workflows/sync-modules.yml`

Final structure:
```
study-calendar/
├── .github/
│   └── workflows/
│       ├── notion.yml
│       └── sync-modules.yml
├── .gitignore
├── README.md
├── index.html
└── modules.json
```

### 5 · Edit `index.html`

Open `index.html` in the GitHub web editor. Find:
```js
const GITHUB_USER = "YOUR_GITHUB_USERNAME";
```
Replace with your actual username. Commit.

### 6 · Add Secrets

**Settings → Secrets and variables → Actions → New repository secret**

Add two secrets (use the same values as your daily tracker repo):
- `NOTION_TOKEN` → your Notion integration secret
- `NOTION_DATABASE_ID` → the `Study Calendar` database ID from step 2

### 7 · Enable GitHub Pages

**Settings → Pages → Source: „Deploy from a branch" → Branch: `main` / `(root)` → Save**

After 1–2 min your app is live at:
```
https://YOUR-USERNAME.github.io/study-calendar/
```

### 8 · Run the module sync once manually

The cron runs nightly, but for the first time you want modules immediately:

1. **Actions** tab → **Sync Modules from Notion** (left sidebar)
2. Top right: **Run workflow** → **Run workflow** (green button)
3. Wait ~30 sec → refresh the Actions tab → see green checkmark
4. The repo now has a populated `modules.json`

### 9 · Reuse your existing GitHub token

The token from your daily tracker works here too (same `repo` scope). The HTML automatically copies the existing token from `localStorage` if present.

If you don't have one yet, see [this guide](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) — Classic token, scope `repo`.

### 10 · Add to home screen

**iOS (Safari):** open the page → Share → „Add to Home Screen"
**Android (Chrome):** menu → „Add to home screen" / „Install app"

---

## Daily flow

1. Tap the home screen icon
2. Type the task title
3. Pick a date from the calendar
4. Tap the relevant module chip(s)
5. Adjust status (or toggle off for blank status)
6. **Add to Notion**
7. ~30 sec later it shows up in your DB

---

## How modules update

- The cron runs **every night at 03:00 UTC** (~04:00–05:00 CET depending on DST)
- It reads all `Module` multi-select options from your Notion DB
- Writes them to `modules.json` and commits if changed
- The app fetches `modules.json` on every load (with cache-busting)

So: **modules added today appear in the app tomorrow morning.**

If you need them sooner, run the sync manually (step 8 above).

---

## Troubleshooting

| Symptom | Cause |
|---|---|
| Status `404` after submit | Username in `index.html` wrong, or repo name not `study-calendar` |
| Status `401` | GitHub token expired or missing `repo` scope |
| Workflow fails: „property does not exist" | DB property names don't match exactly (case-sensitive!) |
| Workflow fails: „Could not find database" | Integration not connected to the `Study Calendar` DB (step 1) |
| Workflow fails on status: „option does not exist" | Your Notion `Status` doesn't have one of the expected option names |
| Modules list empty in app | Cron hasn't run yet, or `Module` property isn't `multi_select` type |
| „Failed to load modules" | `modules.json` not committed, or repo path wrong |

---

## Reset token

Tap the footer („STUDY · CALENDAR · v1") → confirm → token is cleared.
