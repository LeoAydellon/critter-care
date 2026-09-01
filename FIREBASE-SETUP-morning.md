# Tend → LIVE (cloud sync) — Morning Setup Checklist
**For Leo, ~10–15 min, first thing. Do this WITH Athena. Everything free.**

Goal: create a free Firebase project so Tend's data + photos live in the cloud and sync to everyone who has a farm's link.

---

## Step 1 — Create the Firebase project (~3 min)
1. Go to **https://console.firebase.google.com** and sign in with your Google account (leo@studiobeltran.com).
2. Click **"Create a project"** (or "Add project").
3. Name it **`tend-app`** → Continue.
4. Google Analytics: **toggle OFF** (we don't need it) → Create project → wait → Continue.

## Step 2 — Add a Web App to get the keys (~2 min)
1. On the project home, click the **`</>`** (Web) icon ("Add app").
2. Nickname: **`tend`** → **Register app** (skip "Firebase Hosting" checkbox).
3. It shows a `firebaseConfig = { apiKey: "...", ... }` block. **Copy that whole block** — this is what Athena needs. Paste it to Athena in chat. (These keys are safe to be public for this kind of app.)

## Step 3 — Turn on Firestore (the database) (~2 min)
1. Left menu → **Build → Firestore Database** → **Create database**.
2. Choose **Start in production mode** → pick location **`us-west` (or nearest)** → Enable.
3. Go to the **Rules** tab, replace the rules with the block Athena gives you (in `firestore.rules` in this folder) → **Publish**.

## Step 4 — Turn on Storage (for photos) (~2 min)
1. Left menu → **Build → Storage** → **Get started**.
2. Start in production mode → same location → Done.
3. **Rules** tab → paste the block from `storage.rules` in this folder → **Publish**.

## Step 5 — Give the keys to Athena
- Paste the `firebaseConfig` block into chat. Athena drops it into the app, deploys, and we test on your phone + Raphael's together.

---

### What each thing is (plain English)
- **Firebase** = Google's free "backend in a box" — it holds your data and photos online.
- **Firestore** = the live database (the areas, tasks, who-marked-done).
- **Storage** = where the actual photo files live (photos are too big for the database).
- **The config keys** = the app's "address" to find YOUR Firebase. Safe to be public.
- **Rules** = the lock on the door (who can read/write). We use a simple "anyone with a farm's link can use that farm, but can't touch other farms."

### After setup, how it works
- Each **farm** = its own link (`.../critter-care/?farm=ravens-roost`). Share that link → everyone syncs that farm, photos and all, live.
- New person opens the plain link → makes their OWN farm → their own private setup. Public-safe.
