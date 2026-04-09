# Soccer-Attendance-Tracker

A Next.js app with Google sign-in, role-based access (admin/viewer), and a Firestore database.
Deploys for free on Vercel + Firebase.

---

## What you need before starting

- A Google account
- Node.js 18+ installed on your computer
- A free [Vercel](https://vercel.com) account
- A free [Firebase](https://firebase.google.com) account

---

## Step 1 — Set up Firebase (database)

1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. Click **Add project** → name it (e.g. "soccer-attendance") → click through the steps
3. In the left sidebar go to **Firestore Database** → **Create database**
   - Choose **Start in production mode**
   - Pick a region close to you (e.g. `us-central1`)
4. Go to **Project Settings** (gear icon) → **General** tab
   - Scroll to "Your apps" → click the **</>** (web) icon
   - Register the app (any nickname) — you'll get a config object like:
// Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";
import { getAnalytics } from "firebase/analytics";
// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
// For Firebase JS SDK v7.20.0 and later, measurementId is optional
const firebaseConfig = {
  apiKey: "AIzaSyBRTtQ6KnbGMnxDrTt9UuR-I036eKPP7Uk",
  authDomain: "soccer-attendance-e8ae5.firebaseapp.com",
  projectId: "soccer-attendance-e8ae5",
  storageBucket: "soccer-attendance-e8ae5.firebasestorage.app",
  messagingSenderId: "958663286597",
  appId: "1:958663286597:web:c6b7606dd6531533c4e82c",
  measurementId: "G-JKPKG8DR6N"
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);
const analytics = getAnalytics(app);

5. Go to **Project Settings** → **Service accounts** tab
   - Click **Generate new private key** → confirm → a JSON file downloads
   - Open it and find: `project_id`, `client_email`, `private_key`

6. Deploy Firestore security rules:
   - Install Firebase CLI: `npm install -g firebase-tools`
   - Run: `firebase login`
   - Run: `firebase deploy --only firestore:rules --project YOUR_PROJECT_ID`

---

## Step 2 — Set up Google OAuth

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Select your Firebase project from the dropdown at the top
3. Go to **APIs & Services** → **Credentials**
4. Click **+ Create Credentials** → **OAuth client ID**
   - Application type: **Web application**
   - Name: anything (e.g. "Soccer Attendance")
   - Under **Authorised JavaScript origins**, add:
     - `http://localhost:3000`
     - `https://your-vercel-app.vercel.app` *(add after Vercel deploy)*
   - Under **Authorised redirect URIs**, add:
     - `http://localhost:3000/api/auth/callback/google`
     - `https://your-vercel-app.vercel.app/api/auth/callback/google` *(add after Vercel deploy)*
5. Click **Create** → copy your **Client ID** and **Client Secret**

---

## Step 3 — Configure environment variables locally

Copy the example file:
```bash
cp .env.local.example .env.local
```

Fill in `.env.local`:
```env
# Google OAuth
GOOGLE_CLIENT_ID=your_client_id_from_step_2
GOOGLE_CLIENT_SECRET=your_client_secret_from_step_2

# NextAuth — generate a secret with: openssl rand -base64 32
NEXTAUTH_SECRET=a_random_32_char_string
NEXTAUTH_URL=http://localhost:3000

# Firebase client (from Step 1, the web app config)
NEXT_PUBLIC_FIREBASE_API_KEY=AIza...
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=your-project.firebaseapp.com
NEXT_PUBLIC_FIREBASE_PROJECT_ID=your-project
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=your-project.appspot.com
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=1234567890
NEXT_PUBLIC_FIREBASE_APP_ID=1:123:web:abc

# Firebase Admin (from Step 1, the service account JSON)
FIREBASE_PROJECT_ID=your-project
FIREBASE_CLIENT_EMAIL=firebase-adminsdk-xxx@your-project.iam.gserviceaccount.com
FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nYOUR KEY HERE\n-----END PRIVATE KEY-----\n"
```

> **Important:** The `FIREBASE_PRIVATE_KEY` must be in double quotes and use `\n` for newlines (not actual line breaks).

---

## Step 4 — Run locally

```bash
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000)

**First sign-in:** The very first Google account to sign in is automatically made an **admin**. All subsequent users must be added by an admin.

---

## Step 5 — Deploy to Vercel

1. Push the project to GitHub:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   gh repo create soccer-attendance --private --push --source=.
   ```
   *(or create the repo manually on github.com and push)*

2. Go to [vercel.com](https://vercel.com) → **New Project** → import your GitHub repo
3. In the **Environment Variables** section, add all the variables from your `.env.local`
   - Change `NEXTAUTH_URL` to your Vercel URL: `https://your-app.vercel.app`
4. Click **Deploy**

5. After deploy, go back to Google Cloud Console → Credentials → update your OAuth app:
   - Add `https://your-app.vercel.app` to Authorised JavaScript origins
   - Add `https://your-app.vercel.app/api/auth/callback/google` to Authorised redirect URIs

---

## How roles work

| Action | Admin | Viewer |
|--------|-------|--------|
| View sessions & history | ✓ | ✓ |
| Record attendance | ✓ | ✗ |
| Add / remove players | ✓ | ✗ |
| Add / remove users | ✓ | ✗ |
| Change user roles | ✓ | ✗ |
| Delete sessions | ✓ | ✗ |

**Adding users:** Go to Admin → Users tab → enter their Google email address.
They'll be able to sign in immediately. You can set them as Viewer (read-only) or Admin.

---

## Project structure

```
src/
  app/
    page.tsx              ← Record attendance (home)
    history/page.tsx      ← Session history + player rates
    admin/page.tsx        ← Manage players & users
    login/page.tsx        ← Sign-in page
    api/
      auth/[...nextauth]/ ← Google OAuth + session handling
      sessions/           ← GET/POST sessions, DELETE by ID
      players/            ← GET/POST players, DELETE by ID
      admin/              ← GET/POST/PATCH/DELETE users
    globals.css
    layout.tsx
  components/
    Navbar.tsx
    SessionProvider.tsx
  lib/
    firebase.ts           ← Client SDK
    firebase-admin.ts     ← Admin SDK (server only)
  types/index.ts
firestore.rules           ← Lock down direct DB access
firestore.indexes.json    ← Composite indexes for queries
```

---

## Troubleshooting

**"Your account hasn't been added yet"** — You need an admin to add your Google email in Admin → Users.

**Firestore permission denied** — Make sure you deployed `firestore.rules` (Step 1, point 6).

**Private key errors** — Ensure `FIREBASE_PRIVATE_KEY` is wrapped in double quotes and newlines are `\n` not actual breaks.

**OAuth redirect mismatch** — Double-check that both localhost and your Vercel URL are in Google Cloud Console under Authorised redirect URIs.

