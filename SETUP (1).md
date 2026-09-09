# Premier GM League — Remote Draft App Setup

This app is a single static page (`index.html`) that talks to a free Firebase
backend for the shared, hidden bidding data. You host the page (GitHub Pages
works fine, or Firebase Hosting), and Firebase handles the live sync + hiding.

Total setup time: about 15 minutes, one time.

## 1. Create a free Firebase project

1. Go to https://console.firebase.google.com and sign in with any Google account.
2. Click **Add project**, name it something like `pgl-draft`, and finish the wizard
   (you can decline Google Analytics — not needed).

## 2. Turn on Anonymous Authentication

1. In the left sidebar: **Build → Authentication → Get started**.
2. Under **Sign-in method**, enable **Anonymous**. Save.

This is what lets each owner "click their name" with no password — it silently
signs them into a private session tied to their browser.

## 3. Create the Realtime Database

1. Left sidebar: **Build → Realtime Database → Create Database**.
2. Pick any region close to your group.
3. Start in **locked mode** (we'll paste our own rules next).
4. Once created, click the **Rules** tab and paste in the contents of
   `database.rules.json` from this folder, replacing what's there. Click **Publish**.

These rules are what actually hide each team's bid amount from everyone else
until you hit "Reveal" — it's enforced by Firebase itself, not just hidden in
the page.

## 4. Get your config keys

1. Click the gear icon (top left) → **Project settings**.
2. Scroll to **Your apps**, click the **</>** (web) icon to register a new web app.
3. Give it any nickname, skip Firebase Hosting setup here (we'll do that separately below).
4. Firebase will show you a `firebaseConfig` object with keys like `apiKey`,
   `authDomain`, `databaseURL`, etc.

Open `index.html` in this folder, find this block near the top of the `<script>`:

```js
const firebaseConfig = {
  apiKey: "REPLACE_ME",
  authDomain: "REPLACE_ME.firebaseapp.com",
  databaseURL: "https://REPLACE_ME-default-rtdb.firebaseio.com",
  projectId: "REPLACE_ME",
  storageBucket: "REPLACE_ME.appspot.com",
  messagingSenderId: "REPLACE_ME",
  appId: "REPLACE_ME"
};
```

Replace every value with what Firebase showed you, then save the file.

## 5. Host it — two easy options

### Option A: GitHub Pages (what you already know)
1. Create a new GitHub repo, push this folder to it (just `index.html` is required;
   `database.rules.json` and this guide are optional to include for your own reference).
2. In the repo: **Settings → Pages → Deploy from a branch** → pick `main` / root.
3. GitHub gives you a URL like `https://yourname.github.io/pgl-draft/` — that's
   the link you send to all 8 owners.

### Option B: Firebase Hosting (stays in one ecosystem)
1. Install the CLI once: `npm install -g firebase-tools`
2. From this folder: `firebase login`, then `firebase init hosting`
   (pick your project, use this folder as the public directory, say **no** to
   single-page-app rewrite, **no** to GitHub auto-deploys unless you want that).
3. Deploy: `firebase deploy`
4. Firebase gives you a URL like `https://pgl-draft.web.app` — send that link out.

Either way, you can still keep the code in a GitHub repo for backup/version
history even if you deploy through Firebase Hosting.

## 6. Test before draft day

Open the link in two different browsers (or one normal + one incognito window)
and log in as two different teams. Lock a bid as Team A — confirm Team B's tile
just shows "LOCKED" with no number until you hit Reveal. That's the whole point
of this rebuild — worth a 2-minute check together before you're live.

## Notes on the trust model

- There are no passwords — anyone with the link can claim any team by clicking
  its name. That's intentional, per what you asked for. If two people click the
  same team, whoever clicked most recently "owns" it going forward on that
  device. Fine for a private group of friends; not meant to stop a
  determined person from being disruptive on purpose.
- If someone needs to rejoin (closed the tab, refreshed, joined from a new
  device), they just open the link and click their team name again — no code
  needed. Progress is saved permanently in Firebase, not lost on refresh.
- Firebase's free "Spark" tier comfortably covers a single 8-person draft
  session with room to spare — you won't hit any billing prompts for this.
