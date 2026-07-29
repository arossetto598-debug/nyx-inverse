# Nyx InVerse

A series of thoughts brought to you by the voices in my head.

Pages: Home, Poems, Mini-Stories, About, Add Your Voice (private inbox), Author Login, Inbox (admin-only).
Features: 8 switchable colour palettes, scroll-reveal animations, a hand-drawn nav button, and a private message inbox only the author can read.

This is a **static site** (plain HTML/CSS/JS — no build step) plus **Supabase** for the private inbox and author login. That combination is free at this scale and takes about 20 minutes to set up.

---

## 1. Set up Supabase (the private inbox + login)

1. Go to https://supabase.com → **Start your project** → sign in → **New project**.
   - Name: `nyx-inverse` (anything works)
   - Set a database password (save it somewhere safe — you likely won't need it again)
   - Pick the region closest to you → **Create new project** (takes ~2 minutes)
2. Once it's ready, go to **SQL Editor** → **New query**, paste in the contents of `sql/setup.sql` from this project, and click **Run**.
   This creates the `voices` table and locks it down so only `al3ss.rst@gmail.com` can ever read messages back — enforced by the database itself, not just the website.
3. Go to **Project Settings → API**. Copy:
   - **Project URL**
   - **anon public** key
4. Open `js/supabase-config.js` in this project and paste them in:
   ```js
   const SUPABASE_URL = "https://xxxxxxxx.supabase.co";
   const SUPABASE_ANON_KEY = "eyJhbGciOi...";
   ```
   The anon key is safe to expose in front-end code — it can only do what the RLS policies in `setup.sql` allow.
5. Go to **Authentication → Sign In / Providers → Email** and make sure Email (magic link / OTP) is enabled — it is by default.
6. Go to **Authentication → URL Configuration** and set:
   - **Site URL**: your future domain, e.g. `https://nyxinverse.com` (you can update this later once your domain is live — use your Netlify/Vercel URL for now)
   - **Redirect URLs**: add both `https://YOUR-DOMAIN/admin.html` and your temporary hosting URL (e.g. `https://nyx-inverse.netlify.app/admin.html`) so login works before and after you connect the domain.

There are no environment variables in the traditional sense here since it's a static site — the two values above are the only "config" the site needs, and they live directly in `js/supabase-config.js`.

---

## 2. Put the project on GitHub

1. Create a new repository on https://github.com (e.g. `nyx-inverse`), don't initialize it with a README.
2. In this project's folder:
   ```bash
   git init
   git add .
   git commit -m "Nyx InVerse — first light"
   git branch -M main
   git remote add origin https://github.com/YOUR-USERNAME/nyx-inverse.git
   git push -u origin main
   ```

---

## 3. Deploy (Netlify — recommended, free)

1. Go to https://app.netlify.com → **Add new site → Import an existing project**.
2. Connect GitHub, pick the `nyx-inverse` repo.
3. Build settings: leave **Build command** empty and **Publish directory** as `/` (root) — it's a static site, nothing to build.
4. Click **Deploy site**. You'll get a live URL like `https://random-name-123.netlify.app`.
5. Optional: rename it under **Site settings → General → Change site name**.

(Vercel works identically: **Add New → Project → Import**, framework preset **Other**, no build command, output directory `/`.)

---

## 4. Connect your custom domain

In Netlify:
1. **Site settings → Domain management → Add a domain**, type your domain (e.g. `nyxinverse.com`).
2. Netlify shows you DNS records to add. Two common paths:
   - **Using Netlify DNS (simplest):** Netlify gives you 4 nameservers — go to your domain registrar (GoDaddy, Namecheap, etc.), find **Nameservers**, and replace them with Netlify's. Propagation takes anywhere from a few minutes to 24 hours.
   - **Keeping your current DNS provider:** add an `A` record for the root domain pointing to Netlify's load balancer IP (`75.2.60.5`), and a `CNAME` record for `www` pointing to your `xxxx.netlify.app` address. Netlify shows you the exact values on this screen.
3. Once DNS resolves, Netlify auto-provisions a free HTTPS certificate (Let's Encrypt) — wait for the padlock to turn green under Domain management, usually within an hour.
4. Go back to Supabase → **Authentication → URL Configuration** and update the **Site URL** and **Redirect URLs** to your real domain now that it's live (e.g. `https://nyxinverse.com/admin.html`).

---

## 5. Final checks

- [ ] Visit the live domain — homepage loads, images/fonts load, no console errors (open DevTools → Console).
- [ ] Click the hand-drawn menu button — sidebar opens/closes, animates smoothly.
- [ ] Open the palette switcher (bottom-right circle) — try a few palettes, refresh the page, confirm your choice persisted.
- [ ] Go to **Add Your Voice**, submit a test message.
- [ ] In Supabase → **Table Editor → voices**, confirm the message arrived.
- [ ] Go to **Author Login**, enter `al3ss.rst@gmail.com`, check that inbox for the magic link, click it.
- [ ] Confirm you land on **Inbox** and can see the test message.
- [ ] Try entering a *different* email on the login page — confirm it's rejected before any link is even sent.
- [ ] (Optional, to be thorough) Sign up with a second, different email directly through Supabase's own auth and confirm that even a real session with that email cannot see the inbox — the page signs it out automatically.
- [ ] Test on a phone — nav sidebar, forms, and palette switcher all usable with touch.
- [ ] Check `prefers-reduced-motion` is respected (site settings → accessibility, or your OS's reduce-motion setting) — animations should shorten to near-instant.

---

## Adding new poems or stories

Content is currently written directly into `poems.html` and `stories.html` as HTML cards — open the file, copy an existing `<article class="card ...">` block, and edit the text. No database or build step required. If you later want to add poems from a simple form instead of editing HTML, that's a natural next step and would reuse the same Supabase setup already in place.
