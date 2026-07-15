# studyai

A single-page study tracker for Kartik's **AI/Cloud Solution Architect — Definitive Roadmap 2026**.

The app is hosted as a static `index.html` file, usually through GitHub Pages. Progress is saved locally in the browser first, then optionally synced to Supabase so the same checklist, dates, and notes can be used across devices.

## The Roadmap (v2, July 2026)

The content follows the "Definitive Roadmap" PDF — two source lists merged into one personalised plan:

- **12 domains**, ~429 focused hours, roughly 35 weeks at 10–15 h/week.
- Priorities are `P1` (must do), `P2` (important, after P1s), `P3` (awareness only — read, don't practise).
- Every topic has a **How** (Read / Build / Design / Experiment), a specific **Resource**, and a concrete **Build this project** artefact — mostly tied to Project Sentinel.
- Three domains are entirely new vs the original list: D05 SRE & Deep Observability, D06 Distributed Systems Depth, D07 AI Governance; D11 FinOps is split out as its own domain. Items added vs the original lists carry a blue `★ new` badge.
- A "Recommended Study Sequence" table at the top gives the week-by-week order (start with the D04 golden dataset + RAGAS baseline, not with theory).

### Roadmap v1 → v2 migration

The v2 rewrite replaced the old 11-domain "Master Skill List" (~1,400 h). Because the topics changed, skill IDs are now prefixed `v2-` and the localStorage key was bumped to `kartik_skill_progress_v2`. Old v1 progress is **not** shown in the new tracker (the rows it referred to no longer exist), but it is not deleted either — v1 keys remain in localStorage and in the Supabase `data` JSON, so nothing is lost.

## How It Works

- `index.html` contains the whole app: HTML, CSS, and JavaScript.
- The login screen is client-side only. It keeps casual visitors out, but it is not real security because the username and password are visible in the source code.
- Progress is always saved to `localStorage`, so the tracker works offline on the same browser.
- Cloud sync uses Supabase REST API with a public browser key.
- GitHub Pages only hosts the static file. It does not store progress.
- Supabase stores shared progress in the `progress` table.

## Supabase Setup

1. Create or open a Supabase project.
2. Go to `SQL Editor`.
3. Run this SQL to create the progress table:

```sql
create table if not exists public.progress (
  user_id text primary key,
  data jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);
```

4. Enable Row Level Security:

```sql
alter table public.progress enable row level security;
```

5. Add policies that allow the static app to read and write progress rows:

```sql
create policy "Allow public progress read"
on public.progress
for select
to anon
using (true);

create policy "Allow public progress insert"
on public.progress
for insert
to anon
with check (true);

create policy "Allow public progress update"
on public.progress
for update
to anon
using (true)
with check (true);
```

This setup is simple and works for a private personal tracker, but anyone with the page source can technically use the public Supabase key. Do not store sensitive information in notes.

## Supabase Keys

In Supabase:

1. Go to `Project Settings`.
2. Open `API`.
3. Copy the project URL, for example:

```text
https://your-project-ref.supabase.co
```

4. Copy the public `anon` or `publishable` key.
5. Paste both values into `index.html`:

```js
const SUPABASE_URL = "https://your-project-ref.supabase.co";
const SUPABASE_KEY = "your-public-anon-or-publishable-key";
const SUPABASE_TABLE = "progress";
```

If the app says `Offline` or `Error` after choosing a sync user ID, check that the Supabase URL is correct. A deleted, paused, or wrong project reference will not sync.

## Sync User IDs

After logging in, the app asks for a sync user ID.

Use a simple ID such as:

```text
kartik
onsaurav
```

Use the same ID on every device to sync the same progress.

Allowed characters:

```text
letters, numbers, hyphens, underscores
```

Do not use slashes, spaces, or mixed values such as `onsaurav/Kartik`.

## GitHub Pages Hosting

1. Commit `index.html` and `README.md` to the repository.
2. Push the repository to GitHub.
3. Open the GitHub repository.
4. Go to `Settings` -> `Pages`.
5. Under `Build and deployment`, choose:

```text
Source: Deploy from a branch
Branch: main
Folder: /root
```

6. Save the settings.
7. GitHub will publish the site at a URL like:

```text
https://your-github-username.github.io/studyai/
```

GitHub Pages serves only the static files. Supabase is still required for cross-device sync.

## Troubleshooting

- If login works but sync stays `Offline`, check the browser console for the Supabase error.
- If DNS says the Supabase host does not exist, the `SUPABASE_URL` project reference is wrong or the project no longer exists.
- If Supabase returns `401` or `403`, check the public key and RLS policies.
- If Supabase returns `404`, check that the table is named `progress`.
- If one device does not show another device's progress, make sure both devices use the exact same sync user ID.
- If cloud sync fails, local progress still stays saved in that browser.
