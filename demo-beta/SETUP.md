# PAD Beta Test — Setup Guide

**Time required:** ~15 minutes  
**Who does this:** Omar (one time, before sharing the Netlify link)

---

## Step 1 — Create a free Supabase project

1. Go to [supabase.com](https://supabase.com) → **Start your project** → sign in with GitHub
2. Click **New project**
3. Fill in:
   - **Name:** `pad-beta`
   - **Database password:** anything (save it somewhere)
   - **Region:** pick the closest US region
4. Click **Create new project** — wait ~60 seconds for it to spin up

---

## Step 2 — Get your URL and anon key

1. In your Supabase project → left sidebar → **Project Settings** → **API**
2. Copy two values:
   - **Project URL** — looks like `https://xxxxxxxxxxxx.supabase.co`
   - **anon public** key — long string starting with `eyJ...`

---

## Step 3 — Create the tables (run this SQL)

1. In Supabase → left sidebar → **SQL Editor** → **New query**
2. Paste the entire block below and click **Run**:

```sql
-- Capsules table (mirrors PAD Capsules SharePoint list)
create table capsules (
  id          uuid primary key default gen_random_uuid(),
  style_num   text,
  title       text,
  team        text,
  season      text,
  due_date    date,
  status      text default 'Pending',
  assigned_to text,
  file_url    text,
  file_name   text,
  figma_url   text,
  updated_by  text,
  created_at  timestamptz default now(),
  updated_at  timestamptz default now()
);

-- Rework history (rejections with notes)
create table rework_history (
  id          uuid primary key default gen_random_uuid(),
  capsule_id  uuid references capsules(id),
  rejected_by text,
  stage       text,
  note        text,
  created_at  timestamptz default now()
);

-- Merch tasks
create table merch_tasks (
  id          uuid primary key default gen_random_uuid(),
  team        text,
  route       text,
  assignee    text,
  description text,
  file_url    text,
  file_name   text,
  created_by  text,
  status      text default 'New',
  created_at  timestamptz default now(),
  updated_at  timestamptz default now()
);

-- Enable public read/write (beta test only — no auth)
alter table capsules      enable row level security;
alter table rework_history enable row level security;
alter table merch_tasks   enable row level security;

create policy "public read capsules"       on capsules      for select using (true);
create policy "public write capsules"      on capsules      for all    using (true);
create policy "public read rework"         on rework_history for select using (true);
create policy "public write rework"        on rework_history for all    using (true);
create policy "public read merch_tasks"    on merch_tasks   for select using (true);
create policy "public write merch_tasks"   on merch_tasks   for all    using (true);
```

---

## Step 4 — Create the file storage bucket

1. In Supabase → left sidebar → **Storage** → **New bucket**
2. Name: `capsule-files`
3. Check **Public bucket** → **Create bucket**
4. Click the bucket → **Policies** → **New policy** → **For full customization**
5. Policy name: `public uploads`  
   Allowed operations: SELECT, INSERT, UPDATE  
   Using expression: `true`  
   → **Review** → **Save policy**

---

## Step 5 — Wire the credentials into the HTML file

1. Open `demo-beta/PAD-Demo-Beta.html` in a text editor
2. Find this block near the bottom of the file (search for `YOUR_SUPABASE_URL`):

```javascript
const SUPABASE_URL  = 'YOUR_SUPABASE_URL';
const SUPABASE_ANON = 'YOUR_SUPABASE_ANON_KEY';
```

3. Replace both placeholder values with your real URL and anon key:

```javascript
const SUPABASE_URL  = 'https://xxxxxxxxxxxx.supabase.co';
const SUPABASE_ANON = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
```

4. Save the file.

---

## Step 6 — Deploy to Netlify

1. Go to [netlify.com](https://netlify.com) → sign in → **Add new site** → **Deploy manually**
2. Drag and drop the `PAD-Demo-Beta.html` file onto the Netlify drop zone
3. Netlify gives you a URL like `https://cheerful-llama-abc123.netlify.app`
4. Share that URL with your team

> **Tip:** You can rename the Netlify site to something friendlier like `pad-beta-test` → your URL becomes `https://pad-beta-test.netlify.app`

---

## What testers will see

When a tester opens the link:
1. **Login screen** — pick their name and role
2. **Dashboard** — shows their role view (Designer, Sr. Designer, Merch, Licensing, Manager)
3. **Live Data sections** (teal-bordered cards at the top of key views) — show real shared data
4. **Static demo sections** below — the full demo content remains as a reference

### What each role can do

| Role | Can do |
|---|---|
| Designer / Associate | Add new styles, upload files (.xlsx, PDF, images), submit for Peer Review |
| Sr. Designer | Review Peer queue → Approve (advances to SR Review) or Reject (with note) |
| Sr. Designer | Review SR queue → Approve → Licensing, or Reject |
| Licensing | Review Licensing queue → Approve ✓ or Reject |
| Merch | Create tasks (with file upload), mark tasks complete |
| Manager | See all styles in one table |

---

## Troubleshooting

| Issue | Fix |
|---|---|
| Live sections stay empty after login | Supabase URL/key not wired — re-check Step 5 |
| "Upload failed" error | Storage bucket not public — re-check Step 4 |
| Sync dot turns orange | Supabase unreachable — check browser console for CORS or network error |
| Can't see other testers' changes | Wait up to 30 seconds (auto-refresh interval) or reload the page |

---

## After the test

Once the test is done and you're ready for real deployment:
- Share results with IT
- The production app (`app/PAD-App-V2.html`) already has the same workflow wired to SharePoint + MSAL
- IT admin consent for `Sites.ReadWrite.All` is the only remaining gate
