# TCNCC Lekki Temple Duty Roster — GitHub + Firebase setup

This is a single self-contained webpage (`index.html`) that volunteers use to
sign up for duty slots, and admins use to configure the month and print the
final roster. It stores its data in **Firebase Firestore** (free tier) and is
hosted as a static site on **GitHub Pages** (free).

Total cost: **$0/month** for a church-sized roster. Firebase's free "Spark"
plan includes 50,000 reads, 20,000 writes and 20,000 deletes per day, and
1 GiB of storage — far more than this app will ever use.

You do **not** need to know how to code to follow these steps, just go
carefully in order.

---

## What you're setting up

- **GitHub Pages** hosts the actual webpage people visit (`index.html`).
- **Firebase Firestore** is the database that stores the roster settings and
  everyone's sign-ups, so all volunteers and admins see the same live data.
- The **Firebase project config** you'll paste into `index.html` is *not* a
  secret — it's meant to be public. What actually protects your data is the
  **Firestore Security Rules** file (`firestore.rules`), which you'll paste
  into the Firebase console.

---

## Part 1 — Create your Firebase project

1. Go to <https://console.firebase.google.com/> and sign in with a Google
   account.
2. Click **Add project**. Give it a name (e.g. `tcncc-lekki-roster`) and
   click through the setup screens (you can decline Google Analytics — it's
   not needed).
3. Once the project is created, click the **web icon (`</>`)** on the
   project overview page to register a new web app.
   - Give it a nickname like "Roster website".
   - You do **not** need Firebase Hosting — you're using GitHub Pages
     instead, so you can skip/ignore that checkbox.
4. Firebase will show you a `firebaseConfig` object that looks like this:

   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "tcncc-lekki-roster.firebaseapp.com",
     projectId: "tcncc-lekki-roster",
     storageBucket: "tcncc-lekki-roster.appspot.com",
     messagingSenderId: "123456789012",
     appId: "1:123456789012:web:abcdef1234567890"
   };
   ```

   **Copy this whole block** — you'll paste it into `index.html` in Part 3.

5. In the left sidebar, go to **Build → Firestore Database**, click
   **Create database**.
   - Choose a location close to your congregation (e.g. a European or
     African region if you're in Lagos — pick whichever is closest/lowest
     latency in the list; it can't be changed later).
   - Start in **production mode** (the rules file you'll add in Part 2
     replaces the default locked-down rules anyway).

---

## Part 2 — Lock down the database with the rules file

1. In the Firebase console, go to **Firestore Database → Rules** tab.
2. Delete everything in the editor and paste in the entire contents of the
   `firestore.rules` file provided with this project.
3. Click **Publish**.

This makes sure that even though anyone can write to `config` and
`submissions`, they can only write data shaped like a real roster
entry — not arbitrary junk, and not any other part of your database.

---

## Part 3 — Add your Firebase config to the webpage

1. Open `index.html` in any text editor (Notepad, VS Code, etc.).
2. Find this block near the top of the `<script type="module">` section
   (search for `FIREBASE SETUP`):

   ```js
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
     projectId: "YOUR_PROJECT_ID",
     storageBucket: "YOUR_PROJECT_ID.appspot.com",
     messagingSenderId: "YOUR_SENDER_ID",
     appId: "YOUR_APP_ID"
   };
   ```

3. Replace it with the exact `firebaseConfig` object Firebase showed you in
   Part 1, step 4.
4. Save the file.

---

## Part 4 — Put it on GitHub

1. Go to <https://github.com/new> and create a new repository (e.g.
   `lekki-duty-roster`). It can be public or private — GitHub Pages works
   for both, though a private repo needs a paid plan for Pages, so **public**
   is the simplest free option. (The page itself has no secrets in it, so
   this is safe — see the security note below.)
2. Upload `index.html`, `firestore.rules`, and this `README.md` to the
   repository (on the repo page: **Add file → Upload files**, drag the
   three files in, then **Commit changes**).

---

## Part 5 — Turn on GitHub Pages

1. In your repository, go to **Settings → Pages**.
2. Under "Build and deployment", set **Source** to **Deploy from a branch**.
3. Set **Branch** to `main` and folder to `/ (root)`, then **Save**.
4. GitHub will give you a live URL after a minute or two, usually:

   ```
   https://YOUR-GITHUB-USERNAME.github.io/lekki-duty-roster/
   ```

5. Open that link — you should see the sign-up form load, with a small
   warning banner if the Firebase config wasn't pasted in correctly.

---

## Part 6 — First-time setup on the live site

1. Open the live link. Click the **Admin & Roster** tab.
2. The default PIN is **1234**. Enter it and unlock.
3. **Immediately change the PIN** in the "Admin PIN" card at the bottom, and
   share the new PIN only with the people who should manage the roster.
4. In "Roster Settings", confirm the month/year (already set to
   **October 2026**), click **Generate Sundays for this month**, then
   **Save settings**.
5. Share the GitHub Pages link with your volunteers. Anyone who opens it can
   fill in the **Submit Your Availability** tab — no account or login
   needed on their end.
6. As people sign up, open the **Admin & Roster** tab any time to see
   submissions and the generated roster, and use **Print / Save as PDF**
   when you're ready to publish the final version.

---

## Security, honestly

This app intentionally has no login system, to keep sign-up frictionless
for volunteers. That means:

- **Anyone with the link can submit or overwrite a sign-up** (submitting
  again with the same name updates that entry — this is by design).
- **The admin PIN is not real security.** It's stored in the same open
  database and is readable by anyone who opens their browser's developer
  tools, so it only stops casual/accidental edits, not a determined person.
- **Firestore rules only check the *shape* of writes**, not who's making
  them — they stop garbage data, not misuse by someone who has the link.

For a small church roster this trade-off is usually fine. If you later want
real protection (e.g. only signed-in leaders can access the Admin tab),
the next step would be adding **Firebase Authentication** and rewriting the
rules to check `request.auth`. That's a bigger change — ask if you'd like
help with that upgrade later.

## Costs & limits going forward

- Firebase's free Spark plan (no credit card required) covers this use case
  comfortably. You'll only need to worry about billing if this grows into
  something with thousands of daily visitors.
- GitHub Pages is free for public repositories with no bandwidth concerns
  at this scale.
- If you ever want a custom domain (e.g. `roster.yourchurch.org`) instead of
  the `github.io` address, GitHub Pages supports that for free too — see
  **Settings → Pages → Custom domain**.
