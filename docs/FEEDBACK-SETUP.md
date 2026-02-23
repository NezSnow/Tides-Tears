# Feedback: show for all visitors (Supabase)

Right now, feedback is stored in the browser (localStorage), so only the person who submitted it sees it on their device. To have **everyone** see the same feedback (e.g. on your Vercel app), use Supabase as a small database.

## 1. Create a Supabase project

1. Go to [supabase.com](https://supabase.com) and sign in (free account).
2. Click **New project**, pick a name and password, and create the project.
3. Wait until the project is ready.

## 2. Create the feedback table

1. In the Supabase dashboard, open **SQL Editor**.
2. Run this SQL:

```sql
-- Table for feedback entries
create table public.feedback (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  email text not null,
  message text not null,
  created_at timestamptz default now()
);

-- Let anyone insert feedback (submit form)
alter table public.feedback enable row level security;

create policy "Anyone can insert feedback"
  on public.feedback for insert
  with check (true);

-- Let anyone read feedback (show list to all visitors)
create policy "Anyone can read feedback"
  on public.feedback for select
  using (true);
```

3. Run the query. You should see “Success”.

## 3. Get your project URL and anon key

1. In Supabase, go to **Project settings** (gear icon) → **API**.
2. Copy:
   - **Project URL** (e.g. `https://xxxx.supabase.co`)
   - **anon public** key (long string under “Project API keys”).

## 4. Add them to your site

1. Open **feedback.html** in your project.
2. Find this block near the top (after the Supabase script tag):

```html
<script>
  window.TIDES_FEEDBACK_CONFIG = {
    supabaseUrl: '',
    supabaseAnonKey: ''
  };
</script>
```

3. Paste your values:

```html
<script>
  window.TIDES_FEEDBACK_CONFIG = {
    supabaseUrl: 'https://YOUR_PROJECT.supabase.co',
    supabaseAnonKey: 'YOUR_ANON_KEY_HERE'
  };
</script>
```

4. Save and deploy (e.g. push to Vercel). After that, all visitors will see the same feedback list, and new submissions will be stored in Supabase and visible to everyone.

## Optional: use environment variables on Vercel

To avoid putting the key in the HTML:

1. In Vercel: **Project → Settings → Environment Variables**. Add:
   - `SUPABASE_URL` = your Project URL  
   - `SUPABASE_ANON_KEY` = your anon public key  

2. If you use a build step, you can inject these into `feedback.html` (e.g. replace placeholders during build). For a static deploy with no build, editing the config in **feedback.html** (step 4 above) is the simplest.
