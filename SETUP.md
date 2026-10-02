# Hosting Galactic War yourself

This kit runs the war table on your own site, with the GM tools behind a password.

- **Players and the shop screen:** open the link and see the live galaxy. They don't need an account.
- **GMs:** press **GM sign in** and use an email and password you set up.
- **What the password protects:** the database itself only accepts changes from your GM accounts, and keeps drafts, GM notes and hidden details away from everyone else. Reading the page's code won't get anyone past it.

It uses Google's Firebase for the database and sign-in. The page itself can be hosted on GitHub Pages, deploying automatically from your repository, or on Firebase Hosting. The free plan is enough for a shop. If you ever hit the free limits, the map stops updating until the next day; you are never charged unless you choose to upgrade.

## What's in the kit

| File | What it is |
|---|---|
| `public/index.html` | The war table itself |
| `firestore.rules` | Who can read and change what. You add your GM list here |
| `firestore.indexes.json` | Lookups the history views need |
| `firebase.json` | Tells Firebase where everything is, if you use the Firebase command line |
| `.github/workflows/deploy-pages.yml` | Publishes the page to GitHub Pages on every push |
| `.github/workflows/deploy-firestore-rules.yml` | Optional: sends rule and index changes to Firebase on push |
| `.gitignore` | Keeps backups and keys out of the repository |

## 1. Back up your current map

On the claude.ai version, open **War room → Saves**, scroll to **Back up or move this map**, and press **Download a backup**. Keep the file. It holds every campaign, its draft, history, news, missions, operations, orders, events, battles and GM notes.

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
   - **From GitHub automatically:** see step 8.

A GM who is in Authentication but not in this list cannot sign in to the GM tools, so adding someone always takes both steps.

## 7. Create the indexes

The history and war progress views need three database indexes.

- **From GitHub automatically:** the optional rules workflow in step 8 creates them.
- **In the console:** go to **Firestore Database → Indexes → Composite → Create index** three times, each with Collection ID `events` and query scope **Collection**:

| Field 1 | Field 2 |
|---|---|
| `planets`, Array contains | `at`, Descending |
| `terrs`, Array contains | `at`, Descending |
| `kind`, Ascending | `at`, Descending |

Indexes take a few minutes to build. Until they are ready, planet and territory history panels stay empty.

## 8. Put the site online from GitHub

The kit includes a GitHub workflow that publishes the page to **GitHub Pages** every time you push to `main`. It needs no keys or passwords.

1. Create a repository on GitHub and upload the whole kit folder, including the hidden `.github` folder and `.gitignore`. Make sure `public/index.html` has your Firebase settings from step 3.
2. In the repository, go to **Settings → Pages** and set **Source** to **GitHub Actions**.
3. Push to `main`, or open the **Actions** tab and run **Deploy to GitHub Pages**. When it finishes, the run shows your site's address, usually `https://YOUR-USERNAME.github.io/YOUR-REPOSITORY/`.
4. In Firebase, go to **Authentication → Settings → Authorised domains**, press **Add domain**, and add `YOUR-USERNAME.github.io`. Without this, GM sign-in is refused.

From then on, editing `public/index.html` and pushing to `main` updates the live site within a minute or two.

**What a public repository shows.** On GitHub's free plan, Pages needs a public repository. That's fine for this kit:

- The Firebase settings in `index.html` are meant to be public.
- Your GM user IDs in `firestore.rules` are not passwords; nobody can sign in with them.
- Never commit a campaign backup. Backups hold GM notes and drafts. The included `.gitignore` blocks files named like backups, but take care with anything you rename.

### Security rules: paste them, or deploy them from GitHub too

The page deploys from GitHub. The security rules and indexes live in Firebase, so they need sending there separately, either way:

- **By hand (simplest):** whenever you change your GM list, paste `firestore.rules` into **Firestore Database → Rules** and press **Publish**. Create the indexes once, as in step 7.
- **Automatically (optional):** the kit's second workflow, **Deploy Firestore rules**, sends `firestore.rules` and `firestore.indexes.json` to Firebase whenever you push changes to them. It stays switched off until you set it up:
  1. In the [Google Cloud console](https://console.cloud.google.com), open your Firebase project, go to **IAM & Admin → Service Accounts**, and create a service account (for example `github-rules`).
  2. Give it these roles: **Firebase Rules Admin**, **Cloud Datastore Index Admin** and **Service Usage Consumer**.
  3. Open the account, go to **Keys → Add key → Create new key → JSON**, and download the file.
  4. In your GitHub repository, go to **Settings → Secrets and variables → Actions**:
     - On the **Secrets** tab, add `FIREBASE_SERVICE_ACCOUNT` and paste the whole contents of the key file.
     - On the **Variables** tab, add `FIREBASE_PROJECT_ID` with your project ID (shown in Firebase under **Project settings**).
  5. Delete the downloaded key file from your computer. Never commit it.

  The workflow refuses to deploy while the rules still contain `PASTE-GM-UID-HERE`, so you can't lock yourself out by accident.

### Other ways to host

- **Firebase Hosting, deployed from GitHub:** if you'd rather the site lived on Firebase (address like `galactic-war.web.app`), install the [Firebase command line](https://firebase.google.com/docs/cli) and run `firebase init hosting:github` in the kit's folder. It connects your repository and writes its own workflow. You can then delete `deploy-pages.yml`.
- **Firebase Hosting from your computer:** run `npm install -g firebase-tools`, `firebase login`, `firebase use --add` (pick your project), then `firebase deploy`. This uploads the page, rules and indexes in one go.
- **Netlify or Cloudflare Pages:** both can deploy from your GitHub repository. Set the publish folder to `public`, with no build command. Add the site's domain to Firebase's authorised domains, as in point 4 above.

## 9. Move your campaign across

1. Open your new site.
2. Press **GM sign in** and sign in with a GM account.
3. Press **GM tools**, then go to **War room → Saves → Import a backup**.
4. Choose the backup file from step 1.

The page reloads with your galaxy, drafts and history in place.

## 10. Post updates to Discord (optional)

The hosted site can announce news and galaxy updates in a Discord channel. This only works on your hosted site, not the claude.ai version.

1. In Discord, open the channel's settings (the gear next to its name), go to **Integrations → Webhooks**, press **New Webhook**, then **Copy Webhook URL**.
2. On your site, sign in as a GM and go to **War room → Discord**.
3. Paste the address, choose a name to post as, and tick what to announce:
   - news stories
   - galaxy updates
   - planets captured and invaded
   - space battles
   - major orders
   - galaxy events
   - game results
4. Press **Save**, then **Send a test message**.

Things to know:

- **Per campaign:** each campaign has its own settings, so different campaigns can post to different channels.
- **One sync, one message:** a sync posts a single message covering the galaxy update and anything that went out with it.
- **Captures and invasions:** each planet that changes hands, and each new invasion, gets its own alert with a link to the planet. The 8 most valuable get alerts, with invasions always included, and the rest are listed together in one summary.
- **Story links:** links in a story's message open that story on your site.
- **Pinging a role (optional):** turn on Developer Mode in Discord (**Settings → Advanced**), right-click the role in **Server Settings → Roles**, press **Copy Role ID**, and paste it in.
- **Keep the address private:** treat the webhook address like a password, because anyone who has it can post in that channel. It is stored where only GMs can read it. If it leaks, delete the webhook in Discord and paste in a new one.
- **Where messages come from:** they are sent from the browser of the GM who publishes or syncs, so that GM needs to be online at the time. This also means there is nothing extra to run or pay for.

## Day to day

- **Players and the shop screen:** just open the link. Nobody needs to sign in to see the map or the news.
- **GMs:** press **GM sign in**. Sign out from **War room → Overview → Access**.
- **Forgotten password:** use the **Forgotten password** link on the sign-in window. Firebase emails a reset link.
- **Adding a GM:** add the account in Authentication, add their UID to `firestore.rules`, and publish the rules again.
- **Removing a GM:** take their UID out of the rules and publish. You can also disable or delete the account in Authentication.
- **Saves:** **War room → Saves** keeps named saves of a campaign in your own database, like save slots in a game. Load one to put the campaign back exactly as it was, or load it as a new campaign to branch the war. An autosave is made before every load and once a day when you sync.
- **Backups:** **Download a backup** on the Saves tab gives you an off-site file of every campaign. It's worth keeping one somewhere safe from time to time.
- **Updating the page:** if you get a newer `index.html`, copy your `GALACTIC WAR SETTINGS` block into it, then commit and push. GitHub redeploys it.
- **Adding or removing a GM:** after editing `firestore.rules`, push it (if the rules workflow is set up) or paste it into the console.

## Good to know

- **Separate copies:** the claude.ai version and your hosted site are separate. Changes on one don't appear on the other. Once you've moved, use the hosted site.
- **What's protected:** GM drafts, GM notes, GM-only log entries, the full synced galaxy and snapshots can only be read by your GMs.
- **What players can still see:** what the live galaxy shows them. A planet's original generated values, before any GM edit, are built into the page, so a determined player could still work those out.
