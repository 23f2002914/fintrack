# fintrack — Backend Setup Guide

## Step 1 — Create a Supabase project

1. Go to https://supabase.com → Sign up (free)
2. Click **New Project**
3. Name it `fintrack`, pick any region (Mumbai is closest), set a DB password
4. Wait ~1 min for it to provision

---

## Step 2 — Run the SQL (create all tables)

Go to your Supabase project → **SQL Editor** → paste and run this:

```sql
-- TRANSACTIONS
create table transactions (
  id uuid primary key default gen_random_uuid(),
  date date not null,
  account text not null,
  category text,
  note text,
  amount numeric not null,
  type text,
  transfer boolean default false,
  created_at timestamptz default now()
);

-- BILLS
create table bills (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  amount numeric not null,
  due_date date not null,
  paid boolean default false,
  created_at timestamptz default now()
);

-- DEBTS
create table debts (
  id uuid primary key default gen_random_uuid(),
  direction text not null check (direction in ('owe','owed')),
  person text not null,
  total numeric not null,
  paid numeric default 0,
  monthly numeric default 0,
  note text,
  created_at timestamptz default now()
);

-- ACCOUNT BALANCES
create table balances (
  id uuid primary key default gen_random_uuid(),
  account text not null unique,
  balance numeric not null default 0,
  role text,
  updated_at timestamptz default now()
);

-- Seed initial balances
insert into balances (account, balance, role) values
  ('HDFC', 8919.07, 'income account · main obligations'),
  ('Kotak 811', 251.05, 'daily spending account'),
  ('Slice', 789.42, 'savings · daily interest');

-- Enable Row Level Security (keep data safe but publicly readable since no auth)
alter table transactions enable row level security;
alter table bills enable row level security;
alter table debts enable row level security;
alter table balances enable row level security;

-- Allow all operations (no login required)
create policy "public access" on transactions for all using (true) with check (true);
create policy "public access" on bills for all using (true) with check (true);
create policy "public access" on debts for all using (true) with check (true);
create policy "public access" on balances for all using (true) with check (true);
```

---

## Step 3 — Get your API keys

In Supabase → **Project Settings** → **API**:

- Copy **Project URL** → looks like `https://xyzxyz.supabase.co`
- Copy **anon / public key** → long JWT string

---

## Step 4 — Add keys to index.html

Open `index.html`, find these two lines near the top of the `<script>` section:

```js
const SUPABASE_URL = 'YOUR_SUPABASE_URL';
const SUPABASE_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

Replace with your actual values.

---

## Step 5 — Seed your transaction history

In Supabase → **SQL Editor**, run the seed file `seed.sql` (included in this folder).
This loads all 185 transactions from May–June 2026.

---

## Step 6 — Deploy to Vercel

1. Push this folder to a GitHub repo (make sure it's named `fintrack`)
2. Go to vercel.com → **Add New Project** → import the repo
3. No build settings needed — just deploy
4. Done! Your URL will be `fintrack-xxx.vercel.app`

---

## Folder structure

```
fintrack/
├── index.html      ← the app (deploy this)
├── seed.sql        ← run once in Supabase SQL editor
└── SETUP.md        ← this file
```
