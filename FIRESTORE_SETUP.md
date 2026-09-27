# Rental Home Connect — Firestore Setup Guide

## 1. Firestore Security Rules

Paste these into **Firebase Console → Firestore → Rules tab**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    function isSignedIn() { return request.auth != null; }
    function isAdmin() {
      return isSignedIn()
        && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.isAdmin == true;
    }

    // Users — only the user themselves or an admin can read
    match /users/{uid} {
      allow read: if isSignedIn() && (request.auth.uid == uid || isAdmin());
      allow create: if isSignedIn() && request.auth.uid == uid;
      allow update, delete: if isAdmin();
    }

    // Chats — visitors create their own, anyone signed-in can read their own, admin reads all
    match /chats/{chatId} {
      allow read: if isSignedIn() && (resource.data.visitorUid == request.auth.uid || isAdmin());
      allow create: if isSignedIn();
      allow update: if isSignedIn() && (resource.data.visitorUid == request.auth.uid || isAdmin());
      allow delete: if isAdmin();

      // Messages subcollection
      match /messages/{msgId} {
        allow read: if isSignedIn();
        allow create: if isSignedIn();
        allow update, delete: if isAdmin();
      }
    }

    // Listings, cities, images — public read, admin write
    match /listings/{id} {
      allow read: if true;
      allow write: if isAdmin();
    }
    match /cities/{id} {
      allow read: if true;
      allow write: if isAdmin();
    }
    match /images/{id} {
      allow read: if true;
      allow write: if isAdmin();
    }
    match /leads/{id} {
      allow read: if isAdmin();
      allow create: if true;  // visitors can submit forms
      allow update, delete: if isAdmin();
    }
    match /settings/{docId} {
      allow read: if true;     // public site reads branding/social
      allow write: if isAdmin();
    }
  }
}
```

## 2. Create the Admin User

### Step 1 — Create the Firebase Auth user
1. Go to **Firebase Console → Authentication → Users → Add user**
2. Email: `kingfache@rental.com`
3. Password: `fache123`
4. Click **Add user**
5. **Copy the User UID** that appears in the user list (long string like `kS5yy1GpdFc3...`)

### Step 2 — Add the admin flag in Firestore
1. Go to **Firestore Database → Data**
2. Click **+ Start collection** (or open existing `users` collection)
3. Collection ID: `users`
4. Document ID: *(paste the UID from step 1)*
5. Add field:
   - Field: `email`
   - Type: `string`
   - Value: `kingfache@rental.com`
6. Add another field:
   - Field: `isAdmin`
   - Type: `boolean`
   - Value: `true`
7. Click **Save**

### Step 3 — Test login
1. Open `admin.html` in a browser
2. Sign in with `kingfache@rental.com` / `fache123`
3. You should land in the dashboard.

## 3. Enable Anonymous Auth (for visitor chat)

1. Firebase Console → **Authentication → Sign-in method**
2. Click **Anonymous**
3. Toggle **Enable** → Save

This lets the chat widget auto-sign-in visitors so they get a `uid` before they even type their name.

## 4. Cloudinary Setup

The admin dashboard reuses the Cloudinary account from the support wallet project:
- Cloud name: `dfd1bgdam`
- Upload preset: `musk_chat_unsigned` (must be **unsigned**)

To verify or create the preset:
1. Log into Cloudinary dashboard
2. Settings → Upload → Upload presets
3. Find `musk_chat_unsigned` (or create it: Signing mode = **Unsigned**, Folder = `rhc-properties` or `chat-images`)
4. Save

## 5. Public site URL

The admin dashboard links back to your public site at:
- **https://rental-home-connect.vercel.app**

## 6. Files

- `index.html` — public site (deploy to Vercel)
- `admin.html` — staff portal (deploy alongside index.html on Vercel)
- `FIRESTORE_SETUP.md` — this file

## 7. Deploy

1. Upload both `index.html` and `admin.html` to your Vercel project (or git push to your repo)
2. Once deployed, the public site is at `rental-home-connect.vercel.app` and the admin at `rental-home-connect.vercel.app/admin.html`
3. Triple-click the logo on the public site to open the admin login
