# Live Poll — classroom polling you own

A single HTML page. Students scan a QR code on your slide, tap an answer on
their phone, and the bars on the projector move as the votes land. No student
accounts, no per-seat license, no data leaving your own Firebase project.

Three files:

| file | what it is |
|---|---|
| `index.html` | the whole app — teacher console, student view, live results |
| `firestore.rules` | who is allowed to write what (paste into Firebase) |
| `README.md` | this file |

---

## One-time setup (about 10 minutes)

### 1. Create the Firebase project (free)

1. Go to <https://console.firebase.google.com> and **Add project**. Name it
   anything (`uchicago-polls`). Turn Google Analytics **off** — you don't need it.
   No credit card, no billing: the free Spark plan covers a classroom many
   times over.
2. In the left sidebar: **Build → Firestore Database → Create database**.
   Choose **Start in production mode** (the rules in step 3 replace the
   defaults), and pick a region near you — `nam5 (us-central)` is fine.
3. **Build → Authentication → Get started → Anonymous → Enable → Save.**
   This is what lets a student vote without logging in: their browser silently
   gets a throwaway identity, which is how the app stops one phone from
   voting twenty times.

### 2. Paste your config into `index.html`

In the Firebase console: **⚙ Project settings → Your apps → Web (`</>`)**.
Register an app (any nickname, skip Firebase Hosting). It shows you a
`firebaseConfig` object. Copy it and replace the `PASTE_ME` block near the
top of `index.html`.

Those values are **not secrets** — they ship in every Firebase web app and
are meant to be public. Access is controlled by the rules, not by hiding them.

### 3. Publish the security rules

In the console: **Firestore Database → Rules** tab. Delete what's there,
paste the entire contents of `firestore.rules`, click **Publish**.

These rules enforce four things:

- a student can only write **their own** vote document, never anyone else's;
- votes are only accepted while **you** have voting open;
- only the person who created a poll can edit its question or clear its votes;
- nobody can delete someone else's poll.

### 4. Put it on GitHub Pages

```bash
# in a new folder
git init
git add index.html firestore.rules README.md
git commit -m "Live poll app"
git branch -M main
git remote add origin https://github.com/XuanyaoLiuLab/livepoll.git
git push -u origin main
```

Then on GitHub: **Settings → Pages → Source: Deploy from a branch →
`main` / `root` → Save.** A minute later the site is live at

```
https://xuanyaoliulab.github.io/livepoll/
```

The repo must be **public** for free GitHub Pages (that's fine — there's no
secret in it, and student votes live in Firestore, not in the repo).

---

## Using it in class

1. Open the site on the machine you'll teach from. Click **Start a new poll**.
2. Type the question and the options, click **Save question**.
3. **Bookmark this page.** The URL ends in `#t/ABCD` — that's your console for
   this poll. Create the poll on the machine you'll actually present from:
   editing rights are tied to the browser that made it.
4. Put the QR code on screen (or click **Present** for a projector-sized view
   with a small QR in the corner for latecomers).
5. Click **Open voting**. Bars move live as answers arrive.
6. Click **Close voting** when you're done — students' phones switch to
   showing the results, which is a nice moment for the discussion.
7. **Clear votes** re-runs the same question with a different section.
   **New poll** starts a fresh question with a fresh code.

Students can change their answer while voting is open, which is what you want
for a think-pair-share: poll, discuss, re-poll.

---

## Good to know

**Cost.** Firestore's free tier gives 50,000 document reads and 20,000 writes
a day. A 40-student poll with live results costs roughly 40 writes and a few
thousand reads. You would need dozens of large lectures per day to approach
the limit.

**Anonymity.** You see counts, never names. The anonymous Firebase uid is
opaque and isn't shown anywhere in the interface. If a student clears their
browser data, they can vote again — this is a temperature check, not an exam.

**Who can open your console.** Anyone who types your `#t/ABCD` URL sees the
console layout, but the security rules won't let them change anything: only
the browser that created the poll can edit or clear it. Just don't project the
console URL — project the QR code or use Present mode.

**A student on a laptop and a phone** counts as two votes. Unavoidable
without logins, and not worth fixing for classroom use.

**If something looks broken**, the two usual causes are: Anonymous sign-in
wasn't enabled in Authentication, or the rules weren't published. The page
tells you which when a call fails.

**Reusing a question next quarter.** Open the old `#t/ABCD` bookmark and hit
Clear votes — the question is still there.
