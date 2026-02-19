# Umami Analytics Setup Plan

## What's already done (by Claude)

The Hugo site has been wired up to load Umami's tracking script. Three changes were made:

| File | What it does |
|------|-------------|
| `layouts/_partials/analytics.html` | Renders the Umami `<script>` tag (only if both config params are set) |
| `layouts/baseof.html` | Overrides the theme's baseof to include the analytics partial |
| `hugo.toml` | Added `umamiScript` and `umamiWebsiteId` params (placeholder values) |

The script won't load on the live site until you replace the placeholders with real values.

---

## What you need to do

### Step 1: Create the database (Supabase)

1. Go to [supabase.com](https://supabase.com) and sign up (free tier: 500MB, 2 projects)
2. Create a new project — pick a strong database password and save it
3. Once the project is ready, go to **Settings > Database > Connection string**
4. Copy the **Transaction pooler** URI — it looks like:
   ```
   postgresql://postgres.[ref]:[password]@aws-0-[region].pooler.supabase.com:6543/postgres
   ```
5. Keep this string handy for the next step

### Step 2: Deploy Umami to Vercel

1. Go to [github.com/umami-software/umami](https://github.com/umami-software/umami) and **Fork** the repo to your GitHub account
2. Go to [vercel.com](https://vercel.com), sign in with GitHub
3. Click **New Project** > import your Umami fork
4. Before deploying, add one environment variable:
   - **Name:** `DATABASE_URL`
   - **Value:** the Supabase connection string from Step 1
5. Click **Deploy** — Umami auto-creates its tables on first boot
6. Once deployed, note your Umami URL: `https://your-project.vercel.app`

### Step 3: Configure Umami

1. Open your Umami URL in a browser
2. Log in with default credentials: **admin** / **umami**
3. **Change the password immediately** (Settings > Profile)
4. Go to **Settings > Websites > Add website**
5. Enter `pedrogasparotti.com` as the domain
6. After adding, click the website entry and copy the **Website ID** (a UUID like `a1b2c3d4-...`)

### Step 4: Update hugo.toml

Open `hugo.toml` and replace the two placeholder values:

```toml
umamiScript = "https://your-project.vercel.app/script.js"   # <-- your actual Vercel URL
umamiWebsiteId = "a1b2c3d4-..."                              # <-- your actual Website ID
```

### Step 5: Commit and push

```bash
git add hugo.toml layouts/
git commit -m "Add Umami analytics tracking"
git push origin main
```

GitHub Actions will auto-deploy the site.

---

## How to verify it works

1. **Check the HTML source:** Visit `pedrogasparotti.com`, right-click > View Page Source, search for `script.js` — you should see the Umami script tag
2. **Check the Umami dashboard:** Open your Umami URL — your visit should appear in real-time
3. **If the script tag is missing:** Make sure both `umamiScript` and `umamiWebsiteId` in `hugo.toml` are non-empty and the site was redeployed

---

## Architecture recap

```
Visitor --> pedrogasparotti.com (GitHub Pages)
                |
                | <script src=".../script.js">
                v
            Umami (Vercel free tier)
                |
                v
            PostgreSQL (Supabase free tier)
```

All three services are free tier. No credit card required.

---

## Cost & limits

| Service | Free tier limit | What happens if exceeded |
|---------|----------------|------------------------|
| Supabase | 500MB storage, 2 projects | Paused after 1 week inactivity (wake manually) |
| Vercel | 100GB bandwidth/month, serverless function limits | Builds stop until next billing cycle |
| GitHub Pages | 100GB bandwidth/month | Site goes offline until next cycle |

For a personal portfolio site, you'll likely never hit any of these limits.
