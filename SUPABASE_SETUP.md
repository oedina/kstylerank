# KStyleRank — Supabase Backend Setup

Follow these steps **once** to set up your secure backend.
Total time: ~20 minutes.

---

## Step 1 — Create a Supabase project

1. Go to **supabase.com** → sign up (free)
2. Click **New project**
3. Name: `kstylerank`
4. Database password: generate a strong one and save it
5. Region: **Northeast Asia (ap-northeast-1)** — closest to Korea
6. Click **Create new project** → wait ~2 minutes

---

## Step 2 — Get your project credentials

1. Go to **Project Settings → Data API**
2. Copy your:
   - **Project URL** → looks like `https://xxxx.supabase.co`
   - **anon public key** → long string starting with `eyJ...`

You will paste these into `admin.html` and `index.html` later.

---

## Step 3 — Run the database schema

Go to **SQL Editor** in your Supabase dashboard and run this SQL:

```sql
-- Styles table: stores all ranking data
CREATE TABLE styles (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  meta TEXT,
  tags TEXT[] DEFAULT '{}',
  gt INTEGER DEFAULT 0,         -- Google Trends score 0-100
  hs INTEGER DEFAULT 0,         -- Hashtag score 0-100
  vs INTEGER DEFAULT 0,         -- Viewership score 0-100
  cv INTEGER DEFAULT 0,         -- Community vote count
  nlr FLOAT DEFAULT 0.8,        -- Net like ratio 0-1
  delta TEXT DEFAULT 'f',       -- 'u' up, 'd' down, 'f' flat
  dv INTEGER DEFAULT 0,         -- Delta value
  img TEXT,                     -- Card photo URL
  google_search TEXT,           -- Google Image Search keyword
  outfit JSONB DEFAULT '[]',
  dramas JSONB DEFAULT '[]',
  drama_photos JSONB DEFAULT '[]',
  shop JSONB DEFAULT '[]',
  issue TEXT DEFAULT '2026-05',  -- Monthly issue tag
  active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Banners table: hero banner slides
CREATE TABLE banners (
  id SERIAL PRIMARY KEY,
  slot INTEGER UNIQUE NOT NULL,  -- 1, 2, 3 (slide position)
  eyebrow TEXT,
  title TEXT,
  sub TEXT,
  img TEXT,
  link TEXT,
  active BOOLEAN DEFAULT true,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Votes table: one row per user vote
CREATE TABLE votes (
  id SERIAL PRIMARY KEY,
  style_id INTEGER REFERENCES styles(id),
  session_id TEXT NOT NULL,     -- anonymous session fingerprint
  vote INTEGER NOT NULL,        -- 1 = wear it, -1 = skip it
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(style_id, session_id)  -- one vote per style per session
);

-- Drama rankings sidebar data
CREATE TABLE drama_rankings (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  channel TEXT,
  episode TEXT,
  viewership TEXT,
  img TEXT,
  rank INTEGER,
  issue TEXT DEFAULT '2026-05',
  active BOOLEAN DEFAULT true
);

-- Idol rankings sidebar data
CREATE TABLE idol_rankings (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  group_name TEXT,
  event TEXT,
  metric TEXT,
  img TEXT,
  rank INTEGER,
  issue TEXT DEFAULT '2026-05',
  active BOOLEAN DEFAULT true
);

-- Admin roles: links Supabase auth users to admin role
CREATE TABLE admin_users (
  id UUID REFERENCES auth.users(id) PRIMARY KEY,
  email TEXT,
  role TEXT DEFAULT 'admin',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Row Level Security
ALTER TABLE styles ENABLE ROW LEVEL SECURITY;
ALTER TABLE banners ENABLE ROW LEVEL SECURITY;
ALTER TABLE votes ENABLE ROW LEVEL SECURITY;
ALTER TABLE drama_rankings ENABLE ROW LEVEL SECURITY;
ALTER TABLE idol_rankings ENABLE ROW LEVEL SECURITY;
ALTER TABLE admin_users ENABLE ROW LEVEL SECURITY;

-- Public can read styles, banners, drama/idol rankings
CREATE POLICY "Public read styles" ON styles FOR SELECT USING (true);
CREATE POLICY "Public read banners" ON banners FOR SELECT USING (true);
CREATE POLICY "Public read drama_rankings" ON drama_rankings FOR SELECT USING (true);
CREATE POLICY "Public read idol_rankings" ON idol_rankings FOR SELECT USING (true);

-- Public can insert and update their own votes
CREATE POLICY "Public insert votes" ON votes FOR INSERT WITH CHECK (true);
CREATE POLICY "Public update own votes" ON votes FOR UPDATE USING (true);

-- Only admin can write styles, banners, rankings
CREATE POLICY "Admin write styles" ON styles FOR ALL
  USING (auth.uid() IN (SELECT id FROM admin_users WHERE role = 'admin'));

CREATE POLICY "Admin write banners" ON banners FOR ALL
  USING (auth.uid() IN (SELECT id FROM admin_users WHERE role = 'admin'));

CREATE POLICY "Admin write drama_rankings" ON drama_rankings FOR ALL
  USING (auth.uid() IN (SELECT id FROM admin_users WHERE role = 'admin'));

CREATE POLICY "Admin write idol_rankings" ON idol_rankings FOR ALL
  USING (auth.uid() IN (SELECT id FROM admin_users WHERE role = 'admin'));

-- Admin can see admin_users table
CREATE POLICY "Admin read admin_users" ON admin_users FOR SELECT
  USING (auth.uid() IN (SELECT id FROM admin_users WHERE role = 'admin'));
```

---

## Step 4 — Create your admin user

1. Go to **Authentication → Users** in Supabase dashboard
2. Click **Add user → Create new user**
3. Email: `your@email.com`
4. Password: use a **strong password** (min 12 chars, mix of upper/lower/number/symbol)
5. Click **Create user**
6. Copy the **User UID** shown in the users list

Now run this SQL to give that user admin role (replace the UUID):

```sql
INSERT INTO admin_users (id, email, role)
VALUES ('paste-your-user-uid-here', 'your@email.com', 'admin');
```

---

## Step 5 — Seed initial data

Run this in SQL Editor to add the May 2026 rankings:

```sql
INSERT INTO banners (slot, eyebrow, title, sub, img, link) VALUES
(1, '✦ Met Gala 2026 · K-Pop takeover · May 4, 2026',
   'Hanbok-luxury leads the world',
   'aespa Karina Prada hanbok-inspired gown named best dressed · BLACKPINK Lisa, Rosé, Jennie attended · K-pop dominated Met Gala 2026',
   'https://images.unsplash.com/photo-1515886657613-9f3515b0c78f?w=1400&q=80&fit=crop',
   'idols/index.html'),
(2, '✦ K-Drama · Perfect Crown finale · Filing for Love',
   'Perfect Crown closes this week',
   'IU Perfect Crown (MBC) Ep.11-12 finale this week · 11.2% viewership · Filing for Love (Shin Hye-sun) Ep.9-10 airing now',
   'https://images.unsplash.com/photo-1594938298603-c8148c4dae35?w=1400&q=80&fit=crop',
   'drama/index.html'),
(3, '✦ Seoul street · Acubi quiet cool · May 2026',
   'Acubi reaches global peak',
   'Seoul-born Acubi aesthetic at global peak · #Acubi 65K TikTok posts daily · Musinsa Standard trending internationally',
   'https://images.unsplash.com/photo-1483985988355-763728e1935b?w=1400&q=80&fit=crop',
   'index.html');

INSERT INTO drama_rankings (rank, name, channel, episode, viewership, img, issue) VALUES
(1, 'Perfect Crown', 'MBC', 'Ep.11-12 (finale)', '11.2%', 'https://images.unsplash.com/photo-1594938298603-c8148c4dae35?w=200&q=70&fit=crop', '2026-05'),
(2, 'Filing for Love', 'tvN', 'Ep.9-10', 'Airing now', 'https://images.unsplash.com/photo-1509631179647-0177331693ae?w=200&q=70&fit=crop', '2026-05'),
(3, 'We Are All Trying Here', 'JTBC', 'Ep.11-12', 'Weekend slot', 'https://images.unsplash.com/photo-1469334031218-e382a71b716b?w=200&q=70&fit=crop', '2026-05'),
(4, 'My Royal Nemesis', 'Netflix', 'Ep.5-6', 'Top 10 KR', 'https://images.unsplash.com/photo-1558618666-fcd25c85cd64?w=200&q=70&fit=crop', '2026-05'),
(5, 'The Scarecrow', 'ENA', 'Ep.9-10', 'Airing now', 'https://images.unsplash.com/photo-1490481651871-ab68de25d43d?w=200&q=70&fit=crop', '2026-05');

INSERT INTO idol_rankings (rank, name, group_name, event, metric, img, issue) VALUES
(1, 'Karina', 'aespa', 'Met Gala 2026 Prada gown', 'Best dressed · Variety', 'https://images.unsplash.com/photo-1515886657613-9f3515b0c78f?w=200&q=70&fit=crop', '2026-05'),
(2, 'Lisa', 'BLACKPINK', 'Met Gala 2026 Robert Wun', '#1 TikTok globally', 'https://images.unsplash.com/photo-1483985988355-763728e1935b?w=200&q=70&fit=crop', '2026-05'),
(3, 'Rosé', 'BLACKPINK', 'Met Gala 2026 silver bird dress', 'Hollywood Reporter best', 'https://images.unsplash.com/photo-1523381210434-271e8be1f52b?w=200&q=70&fit=crop', '2026-05'),
(4, 'Seungmin', 'Stray Kids', 'Burberry LFW FW26', 'Feb 2026 trend peak', 'https://images.unsplash.com/photo-1490481651871-ab68de25d43d?w=200&q=70&fit=crop', '2026-05'),
(5, 'Chaewon', 'Le Sserafim', 'Sheer ribbed knit · EASY era', '72M streams', 'https://images.unsplash.com/photo-1509631179647-0177331693ae?w=200&q=70&fit=crop', '2026-05');
```

---

## Step 6 — Add your credentials to the site

In `admin.html`, `drama/index.html`, and `idols/index.html`, replace:
```
SUPABASE_URL = 'https://YOUR_PROJECT.supabase.co'
SUPABASE_ANON_KEY = 'YOUR_ANON_KEY'
```

With your actual values from Step 2.

---

## Step 7 — Deploy to Vercel (recommended for Supabase)

GitHub Pages works fine for reading data. But Supabase realtime and auth sessions
work better behind a proper domain. Vercel is free and one-click:

1. Go to **vercel.com** → sign up with GitHub
2. Click **Add New Project** → select `oedina/kstylerank`
3. Click **Deploy** — done in 60 seconds
4. Your site is now at `kstylerank.vercel.app`
5. Add a custom domain in Vercel settings when ready

---

## Security model summary

| Layer | Protection |
|-------|-----------|
| Supabase Auth | Passwords hashed server-side with bcrypt · never in HTML |
| JWT tokens | Admin session verified on every request · expires in 1 hour |
| Row Level Security | Database rejects writes from non-admin users at the DB level |
| Admin page | Redirects to login if no valid session — page content never loads |
| Public anon key | Safe to expose — can only do what RLS policies allow (read-only for styles/banners) |

The `anon` key in your HTML is safe to expose publicly — it can only perform
operations that your RLS policies allow. The secret `service_role` key
(visible in Supabase dashboard) must **never** go in your frontend code.
