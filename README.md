# Speaking Partner — Setup Guide

## What you need before starting
- GitHub account (github.com)
- Supabase account (supabase.com) — free
- Railway account (railway.app) — free
- Vercel account (vercel.com) — free
- Anthropic API key (console.anthropic.com) — free credits on signup

---

## STEP 1 — Supabase setup (database)

1. Go to supabase.com → New project → give it a name → set a password → Create
2. Wait 1-2 min for project to start
3. Left sidebar → SQL Editor → New query → paste this and click Run:

```sql
create table sessions (
  id uuid primary key,
  persona text not null,
  created_at timestamptz default now()
);

create table messages (
  id uuid default gen_random_uuid() primary key,
  session_id uuid references sessions(id),
  role text not null,
  content text not null,
  created_at timestamptz default now()
);

alter table sessions enable row level security;
alter table messages enable row level security;

create policy "allow all on sessions" on sessions for all using (true) with check (true);
create policy "allow all on messages" on messages for all using (true) with check (true);
```

4. Left sidebar → Project Settings → API
   - Copy "Project URL" → this is your SUPABASE_URL
   - Copy "anon public" key → this is your SUPABASE_ANON_KEY
   - Save both somewhere

---

## STEP 2 — Anthropic API key

1. Go to console.anthropic.com → Sign up / Login
2. Left sidebar → API Keys → Create Key
3. Copy the key (starts with sk-ant-...)
4. Save it somewhere safe — you won't see it again

---

## STEP 3 — Push code to GitHub

On your laptop, open terminal:

```bash
# Go inside the project folder
cd speaking-agent

# Initialize git
git init

# Add all files
git add .

# First commit
git commit -m "initial commit"
```

Now go to github.com:
- Click "+" top right → New repository
- Name it: speaking-agent
- Keep it Private (your keys will be in Railway/Vercel, not here, but still keep private)
- Click Create repository

Back in terminal:
```bash
git remote add origin https://github.com/YOUR_USERNAME/speaking-agent.git
git branch -M main
git push -u origin main
```

---

## STEP 4 — Deploy backend on Railway

1. Go to railway.app → Login with GitHub
2. New Project → Deploy from GitHub repo → select "speaking-agent"
3. Railway will detect the project. Click "Add service" → select the repo
4. It will try to build — STOP — first set environment variables:
   - Click on the service → Variables tab → Add these one by one:
     ```
     ANTHROPIC_API_KEY = sk-ant-your-key-here
     SUPABASE_URL = https://xxxxx.supabase.co
     SUPABASE_ANON_KEY = eyxxxxx
     PORT = 3001
     ```
5. Settings tab → Root Directory → type: `server`
6. Settings tab → Start Command → type: `npm start`
7. Click Deploy

Wait 2-3 minutes. Railway will give you a URL like:
`https://speaking-agent-production-xxxx.up.railway.app`

Test it: open that URL + /api/health in browser. You should see `{"ok":true}`

Save this Railway URL — you need it for next step.

---

## STEP 5 — Deploy frontend on Vercel

1. Go to vercel.com → Login with GitHub
2. New Project → Import "speaking-agent" repo
3. IMPORTANT — before deploying:
   - Root Directory → change to: `client`
   - Environment Variables → Add:
     ```
     VITE_API_URL = https://your-railway-url.up.railway.app
     ```
     (paste your Railway URL from Step 4, no trailing slash)
4. Click Deploy

Wait 1-2 minutes. Vercel gives you a URL like:
`https://speaking-agent-xxxx.vercel.app`

---

## STEP 6 — Test on phone and laptop

1. Open the Vercel URL in Chrome (phone or laptop)
2. Pick a role from the list or write your own
3. Click "Start conversation"
4. Tap the mic button — allow microphone permission when asked
5. Speak in English — AI will reply in character + show corrections

---

## Troubleshooting

**Mic not working:** Only works in Chrome. On iPhone it may not work (Apple restriction). Use Android Chrome or laptop Chrome.

**"Connection error":** Railway server is sleeping (free tier sleeps after inactivity). Wait 30 seconds and try again — it will wake up.

**"AI call failed":** Check your ANTHROPIC_API_KEY in Railway variables. Make sure you have credits at console.anthropic.com.

**Corrections not showing:** That's fine — means your English was good enough that AI said "No corrections needed."

---

## Updating the app later

If you change any code:
```bash
git add .
git commit -m "your change description"
git push
```

Railway and Vercel auto-redeploy when you push to GitHub.
