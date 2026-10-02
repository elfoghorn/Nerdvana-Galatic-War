# Hosting Galactic War yourself

This kit runs the war table on your own site, with the GM tools behind a password.

- **Players and the shop screen:** open the link and see the live galaxy. They don't need an account.
- **GMs:** press **GM sign in** and use an email and password you set up.
- **What the password protects:** the database itself only accepts changes from your GM accounts, and keeps drafts, GM notes and hidden details away from everyone else. Reading the page's code won't get anyone past it.

It uses Google's Firebase for the database, sign-in and hosting. The free plan is enough for a shop. If you ever hit the free limits, the map stops updating until the next day; you are never charged unless you choose to upgrade.

## What's in the kit

| File | What it is |
|---|---|
| `public/index.html` | The war table itself |
| `firestore.rules` | Who can read and change what. You add your GM list here |
| `firestore.indexes.json` | Lookups the history views need |
| `firebase.json` | Tells Firebase where everything is |

## 1. Back up your current map

On the claude.ai version, sign in as a GM and open **War room → Campaigns → Download a backup**. Keep the file. It holds every campaign, its draft, history, news, missions, operations, orders, events, battles and GM notes.

## 2. Create a Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and sign in with a Google account.
2. Press **Create a project**, give it a name (for example `galactic-war`), and finish. Google Analytics is not needed.

## 3. Connect the page to your project

1. In the project, press the **</>** (Web) icon to add a web app. Give it any nickname.
2. Firebase shows a block of code containing `const firebaseConfig = { ... }`. Copy just the part in curly brackets, including the brackets.
3. Open `public/index.html` in a text editor. Near the top, find `GALACTIC WAR SETTINGS` and this line:

   ```js
   window.GW_FIREBASE = null;
   ```

   Replace `null` with what you copied, so it looks like this (your values will differ):

   ```js
   window.GW_FIREBASE = {
     apiKey: "AIza...",
     authDomain: "galactic-war.firebaseapp.com",
     projectId: "galactic-war",
     storageBucket: "galactic-war.appspot.com",
     messagingSenderId: "1234567890",
     appId: "1:1234567890:web:abc123"
   };
   ```

   These values are meant to be public; the security rules are what protect your data.

## 4. Turn on sign-in and create the GM accounts

1. Go to **Build → Authentication → Get started**.
2. Under **Sign-in method**, choose **Email/Password**, switch it on, and save.
3. Under **Users**, press **Add user** for each GM, entering their email and a password.
4. Each user now has a **User UID** (a long code). Copy the UID of every GM.

## 5. Create the database

1. Go to **Build → Firestore Database → Create database**.
2. Choose **Start in production mode**.
3. Pick the location closest to your shop (for the UK, `europe-west2`, London). This cannot be changed later.

## 6. Add your GM list and publish the rules

1. Open `firestore.rules` in a text editor.
2. Replace `'PASTE-GM-UID-HERE'` with your GMs' UIDs, each in quotes, separated by commas:

   ```
   return request.auth != null && request.auth.uid in [
     'Xy12AbCdEf...',
     'Pq98ZzKkLm...'
   ];
   ```

3. Publish the rules using either of these:
   - **In the console:** go to **Firestore Database → Rules**, replace everything with the contents of your edited `firestore.rules`, and press **Publish**.
   - **With the Firebase command line:** see step 8.

A GM who is in Authentication but not in this list cannot sign in to the GM tools, so adding someone always takes both steps.

## 7. Create the indexes

The history and war progress views need three database indexes.

- **With the Firebase command line:** these are created for you in step 8.
- **In the console:** go to **Firestore Database → Indexes → Composite → Create index** three times, each with Collection ID `events` and query scope **Collection**:

| Field 1 | Field 2 |
|---|---|
| `planets`, Array contains | `at`, Descending |
| `terrs`, Array contains | `at`, Descending |
| `kind`, Ascending | `at`, Descending |

Indexes take a few minutes to build. Until they are ready, planet and territory history panels stay empty.

## 8. Put the site online

**Option A: Firebase Hosting (recommended).** You need [Node.js](https://nodejs.org) installed. In a terminal, from the kit's folder:

```
npm install -g firebase-tools
firebase login
firebase use --add
firebase deploy
```

When `firebase use --add` asks, pick your project. `firebase deploy` uploads the page, publishes the rules and creates the indexes in one go. It prints your site's address, for example `https://galactic-war.web.app`.

**Option B: any other static host** (Netlify, Cloudflare Pages, GitHub Pages and so on). Upload the contents of the `public` folder. Then:

1. Publish the rules and create the indexes in the console, as in steps 6 and 7.
2. In Firebase, go to **Authentication → Settings → Authorised domains** and add your site's domain. Otherwise GM sign-in is refused.

## 9. Move your campaign across

1. Open your new site.
2. Press **GM sign in** and sign in with a GM account.
3. Press **GM tools**, then go to **War room → Campaigns → Import a backup**.
4. Choose the backup file from step 1.

The page reloads with your galaxy, drafts and history in place.

## Day to day

- **Players and the shop screen:** just open the link. Nobody needs to sign in to see the map or the news.
- **GMs:** press **GM sign in**. Sign out from **War room → Overview → Access**.
- **Forgotten password:** use the **Forgotten password** link on the sign-in window. Firebase emails a reset link.
- **Adding a GM:** add the account in Authentication, add their UID to `firestore.rules`, and publish the rules again.
- **Removing a GM:** take their UID out of the rules and publish. You can also disable or delete the account in Authentication.
- **Backups:** **War room → Campaigns → Download a backup** works on the hosted site too. It's worth doing before big sessions.
- **Updating the page:** if you get a newer `index.html`, copy your `GALACTIC WAR SETTINGS` block into it before uploading.

## Good to know

- **Separate copies:** the claude.ai version and your hosted site are separate. Changes on one don't appear on the other. Once you've moved, use the hosted site.
- **What's protected:** GM drafts, GM notes, GM-only log entries, the full synced galaxy and snapshots can only be read by your GMs.
- **What players can still see:** what the live galaxy shows them. A planet's original generated values, before any GM edit, are built into the page, so a determined player could still work those out.
