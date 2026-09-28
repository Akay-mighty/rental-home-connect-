# Rental Home Connect — Firebase Setup (STEP-BY-STEP)

This guide walks you through every click. Follow in order. Takes ~10 minutes.

---

## STEP 1: Open Firebase Console

1. Go to **https://console.firebase.google.com**
2. Click on your project: **rental-home-connect**
3. You're now on the project dashboard

---

## STEP 2: Paste Firestore Security Rules

This is the MOST IMPORTANT step. Without these rules, sign-in, chat, applications, and favorites will all fail with "permission-denied".

### Where to go:
1. In the left sidebar, click **Firestore Database** (under "Build" → "Firestore Database")
2. Click the **Rules** tab at the top (next to "Data" and "Indexes")
3. You'll see a text editor with some default rules
4. **SELECT ALL** the text in that editor (Ctrl+A or Cmd+A) and **DELETE** it
5. **PASTE** the rules below exactly as they are:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    function isSignedIn() { return request.auth != null; }
    function isAdmin() {
      return isSignedIn()
        && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.isAdmin == true;
    }

    // Admin staff profiles
    match /users/{uid} {
      allow read: if isSignedIn() && (request.auth.uid == uid || isAdmin());
      allow create: if isSignedIn() && request.auth.uid == uid;
      allow update, delete: if isAdmin();
    }

    // RENTER profiles (for visitors who sign up)
    match /renters/{uid} {
      allow read: if isSignedIn() && request.auth.uid == uid;
      allow create, update: if isSignedIn() && request.auth.uid == uid;
      allow delete: if isSignedIn() && request.auth.uid == uid;

      // Saved homes (favorites) — subcollection under each renter
      match /saved/{listingId} {
        allow read: if isSignedIn() && request.auth.uid == uid;
        allow create, delete: if isSignedIn() && request.auth.uid == uid;
      }
    }

    // Live chat — visitors create their own chats, admin reads all
    match /chats/{chatId} {
      allow read: if isSignedIn() && (resource.data.visitorUid == request.auth.uid || isAdmin());
      allow create: if isSignedIn();
      allow update: if isSignedIn() && (resource.data.visitorUid == request.auth.uid || isAdmin());
      allow delete: if isAdmin();

      // Chat messages — subcollection under each chat
      match /messages/{msgId} {
        allow read: if isSignedIn();
        allow create: if isSignedIn();
        allow update, delete: if isAdmin();
      }
    }

    // Property listings — public can read, only admin can write
    match /listings/{id} {
      allow read: if true;
      allow write: if isAdmin();
    }

    // Cities — public read, admin write
    match /cities/{id} {
      allow read: if true;
      allow write: if isAdmin();
    }

    // Image manager records — public read, admin write
    match /images/{id} {
      allow read: if true;
      allow write: if isAdmin();
    }

    // Leads — visitors can submit, only admin can read/manage
    match /leads/{id} {
      allow read: if isAdmin();
      allow create: if true;
      allow update, delete: if isAdmin();
    }

    // Applications — visitors can submit, only admin can read/manage
    match /applications/{id} {
      allow read: if isAdmin();
      allow create: if true;
      allow update, delete: if isAdmin();
    }

    // Settings (branding, social links) — public read, admin write
    match /settings/{docId} {
      allow read: if true;
      allow write: if isAdmin();
    }
  }
}
```

6. Click the **Publish** button (orange button at the top right of the rules editor)
7. Wait for "Rules published successfully" confirmation

---

## STEP 3: Enable Auth Providers

Visitors need to sign in (email/password or Google) and the chat needs anonymous auth.

### Where to go:
1. In the left sidebar, click **Authentication** (under "Build" → "Authentication")
2. Click the **Sign-in method** tab at the top

### Enable Email/Password:
1. Click **Email/Password** in the list
2. Toggle the first switch to **Enable** (Email/Password)
3. Leave "Email link (passwordless)" OFF
4. Click **Save**

### Enable Google:
1. Click **Google** in the list
2. Toggle to **Enable**
3. Select your **support email** from the dropdown (use your Gmail)
4. Click **Save**

### Enable Anonymous:
1. Click **Anonymous** in the list
2. Toggle to **Enable**
3. Click **Save**

### Add your domain to Authorized domains:
1. Still in Authentication, click the **Settings** tab at the top
2. Scroll down to **Authorized domains**
3. Click **Add domain**
4. Type: `rental-home-connect.vercel.app`
5. Click **Add**
6. Also add `localhost` if you want to test locally

---

## STEP 4: Create the Admin User (staff login)

### Create the Firebase Auth user:
1. In Authentication, click the **Users** tab at the top
2. Click **Add user**
3. Email: `kingfache@rental.com`
4. Password: `fache123`
5. Click **Add user**
6. You'll see the user in the list — **copy the User UID** (the long string like `kS5yy1GpdFc3VLlbRiQ4gHH0f8w1`)

### Add the admin flag in Firestore:
1. Go to **Firestore Database** → **Data** tab
2. If you don't see a `users` collection, click **+ Start collection**
   - Collection ID: `users` → click Next
3. Click **+ Add document** (or open the `users` collection)
4. **Document ID**: paste the UID you copied above
5. Add these fields:
   - Field name: `email` → Type: `string` → Value: `kingfache@rental.com`
   - Click **+ Add field**
   - Field name: `isAdmin` → Type: `boolean` → Value: `true`
6. Click **Save**

Now `kingfache@rental.com` can log into the admin dashboard at `rental-home-connect.vercel.app/admin`

---

## STEP 5: Cloudinary Setup (for image uploads)

The code uses Cloudinary cloud name `dbmtqgs3v` and upload preset `rental home connect`.

### Create the upload preset:
1. Go to **https://cloudinary.com/console** and log in
2. Confirm your cloud name (top-right of dashboard) is `dbmtqgs3v`
3. Click **Settings** (gear icon) → **Upload** tab
4. Scroll to **Upload presets** section
5. Click **Add upload preset**
6. Set:
   - **Name**: `rental home connect`
   - **Signing Mode**: **Unsigned** ← this is critical
   - **Folder**: `rhc-properties` (optional)
7. Click **Save**

If the preset name has spaces, that's fine — the code handles it.

### Fallback (if Cloudinary doesn't work):
The code automatically falls back to **Firebase Storage** if Cloudinary fails. To enable Firebase Storage:
1. In Firebase Console, click **Storage** (under "Build" → "Storage")
2. Click **Get started**
3. Accept the default security rules
4. Click **Done**

---

## STEP 6: Deploy to Vercel

1. Push all files (`index.html`, `admin.html`, `vercel.json`, `logo-public.png`) to your GitHub repo
2. Vercel auto-deploys in ~30 seconds
3. Public site: `https://rental-home-connect.vercel.app`
4. Admin dashboard: `https://rental-home-connect.vercel.app/admin`

---

## Quick Reference

| What | Where |
|------|-------|
| Admin login URL | `rental-home-connect.vercel.app/admin` |
| Admin email | `kingfache@rental.com` |
| Admin password | `fache123` |
| Firebase project | `rental-home-connect` |
| Cloudinary cloud name | `dbmtqgs3v` |
| Cloudinary preset | `rental home connect` (unsigned) |
| Contact email | `rentalhomeconnects@gmail.com` |
| TikTok (main) | `@_rentalhomeconnects` |
| TikTok agents | `@_christopherhayes`, `@_ethanhayes1` |

---

## Troubleshooting

**"Missing or insufficient permissions" error:**
→ You haven't published the Firestore rules yet. Go back to Step 2 and click **Publish**.

**Sign-in doesn't persist after refresh:**
→ Check that Email/Password and Anonymous auth providers are enabled (Step 3).

**Google sign-in popup blocked:**
→ The `vercel.json` file includes a `Cross-Origin-Opener-Policy: same-origin-allow-popups` header. If it still fails, try in an incognito window.

**Chat images don't upload:**
→ Either create the Cloudinary preset (Step 5) or enable Firebase Storage as fallback.

**Admin login says "Access denied":**
→ The `users/{uid}` document in Firestore needs `isAdmin: true` (boolean, not string). Go back to Step 4.
