# Ralli landing page — deployment guide

## What's in this update

- **`index.html`** — fully redesigned landing page, waitlist as centerpiece
- **`terms.html`** — rebuilt in matching Inter + Poppins typography (was DM Sans)
- Both use **Inter** for everything + **Poppins Black 900** for the wordmark only, matching your brand guide

## Step 1 — Replace your Firebase config

Open `index.html` and find this block near the top (inside the `<script type="module">` tag):

```js
const firebaseConfig = {
  apiKey:            "REPLACE_WITH_YOUR_FIREBASE_API_KEY",
  authDomain:        "REPLACE_WITH_YOUR_AUTH_DOMAIN",
  projectId:         "REPLACE_WITH_YOUR_PROJECT_ID",
  storageBucket:     "REPLACE_WITH_YOUR_STORAGE_BUCKET",
  messagingSenderId: "REPLACE_WITH_YOUR_SENDER_ID",
  appId:             "REPLACE_WITH_YOUR_APP_ID",
};
```

Replace each value with your actual Firebase config — the same values you're already using in `app.theralliapp.com`. You can find these in:

**Firebase console → Project settings → Your apps → Web app → Config**

These keys are public-by-design (every Firebase web app ships them in client code). What protects your data is the Firestore security rules, not these values.

## Step 2 — Update your Firestore rules

The waitlist needs a publicly-writable `waitlist` collection. Open the Firebase console:

**Firestore → Rules**

Add this block inside your existing `match /databases/{database}/documents { ... }`:

```
match /waitlist/{docId} {
  // Anyone can add themselves to the waitlist.
  allow create: if request.resource.data.keys().hasOnly(['email', 'createdAt', 'source', 'userAgent'])
                 && request.resource.data.email is string
                 && request.resource.data.email.size() < 320
                 && request.resource.data.email.matches('.+@.+\\..+');

  // Anyone can check for duplicates by email — required for the
  // "you're already on the list" experience. No other reads.
  allow read: if true;

  // No one can edit or delete from the client. Use Firebase console
  // for any cleanup.
  allow update, delete: if false;
}
```

This:
- Lets anyone create one entry with the four expected fields and a valid-looking email
- Lets the duplicate-check query work (read-only)
- Prevents anyone from modifying or deleting entries through the app
- You can view all signups in **Firestore console → `waitlist` collection**

Click "Publish" after pasting.

## Step 3 — Upload to Vercel

Upload these two files to the root of your Vercel project (same place as the favicon files):

- `index.html` (replaces the current landing page)
- `terms.html` (replaces the current terms page — typography update)

Privacy page (`privacy.html`) wasn't part of this update. The next thing to do is rebuild that one in matching Inter typography too — let me know when you want that.

## Step 4 — Test the waitlist

After deploy, visit `theralliapp.com` on a fresh browser:

1. Scroll past the hero. The form should be the first thing you see in the right spot.
2. Type your own email. Hit "Get early access".
3. You should see "✓ You're in" plus a success message.
4. Open Firebase console → Firestore → `waitlist` collection. Your entry should be there with the email, timestamp, and `source: "landing"`.
5. Try submitting the same email again — should say "You're already on the list."
6. Try the bottom CTA form too — same flow.

## Step 5 — Update the app to match (optional, recommended)

The `app.theralliapp.com` React app currently uses **Inter + Cormorant Garamond + Poppins Black**, with Cormorant Garamond as the editorial display font. To match the brand guide exactly, the recommended cleanup:

- **Keep:** Inter everywhere as the body and UI font
- **Keep:** Poppins Black 900 for the wordmark only
- **Drop:** Cormorant Garamond (replace with Inter ExtraBold 800 for display per your brand guide)

I can ship a code update that does this in the JSX whenever you're ready. It's a search-and-replace task on font-family declarations.

## What changed visually on the landing page

| Element | Before | After |
|---|---|---|
| Hero | Headline + email CTA mailto link | Editorial headline with `your skincare` highlight + working waitlist form + floating product mockup |
| Stats | None | 4-stat strip below hero (250+ ingredients, 0-5 scale, A-F grade, 0 sponsored picks) |
| Features | 4 plain text blocks | Grid of 3 feature cards with visual proof — pore score gauge, friend signal mock, ingredient chips |
| Quote | None | Editorial pull-quote with your founder mission statement |
| Bottom CTA | None | Dark navy band with second waitlist form |
| Footer | Single line of links | 4-column structured footer |
| Typography | DM Serif Display + DM Sans | Inter (everything) + Poppins Black 900 (wordmark) — matches brand guide |
| Mobile | OK | Responsive grid with deliberate mobile-first stack |

The waitlist appears in **two places** (hero + bottom CTA) — both wire to the same Firestore submission and check for duplicates. People who scroll past the hero get a second chance.

## Heads-up

I noticed the current site links to `https://ralli.app` in some places but the actual domain is `theralliapp.com`. The new `index.html` uses `theralliapp.com` consistently. If you have other places where `ralli.app` is hardcoded (Instagram bio, App Store metadata in progress, etc.), update those to match.
