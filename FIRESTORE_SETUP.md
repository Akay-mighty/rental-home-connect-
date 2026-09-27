# Rental Home Connect — Firebase + Cloudinary Setup

## 1. Firestore Security Rules (UPDATED for renter auth)

Paste into **Firebase Console → Firestore → Rules tab**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    function isSignedIn() { return request.auth != null; }
    function isAdmin() {
      return isSignedIn()
        && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.isAdmin == true;
    }

    // Admin users (staff)
    match /users/{uid} {
      allow read: if isSignedIn() && (request.auth.uid == uid || isAdmin());
      allow create: if isSignedIn() && request.auth.uid == uid;
      allow update, delete: if isAdmin();
    }

    // RENTER profiles (separate from admin users)
    match /renters/{uid} {
      allow read: if isSignedIn() && request.auth.uid == uid;
      allow create, update: if isSignedIn() && request.auth.uid == uid;
      allow delete: if isSignedIn() && request.auth.uid == uid;

      // Saved homes subcollection
      match /saved/{listingId} {
        allow read: if isSignedIn() && request.auth.uid == uid;
        allow create, delete: if isSignedIn() && request.auth.uid == uid;
      }
    }

    // Chats — visitors create their own, admin reads all
    match /chats/{chatId} {
      allow read: if isSignedIn() && (resource.data.visitorUid == request.auth.uid || isAdmin());
      allow create: if isSignedIn();
      allow update: if isSignedIn() && (resource.data.visitorUid == request.auth.uid || isAdmin());
      allow delete: if isAdmin();

      match /messages/{msgId} {
        allow read: if isSignedIn();
        allow create: if isSignedIn();
        allow update, delete: if isAdmin();
      }
    }

    // Public data — anyone can read
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
      allow create: if true;
      allow update, delete: if isAdmin();
    }
    match /settings/{docId} {
      allow read: if true;
      allow write: if isAdmin();
    }
  }
}
```

## 2. Enable Auth Providers (CRITICAL — new for renter auth)

1. **Firebase Console → Authentication → Sign-in method**
2. Enable these providers:
   - **Email/Password** → toggle Enable → Save
   - **Google** → toggle Enable → select your project support email → Save
   - **Anonymous** → toggle Enable → Save (for visitor chat)
3. Add your domain `rental-home-connect.vercel.app` to **Authorized domains** (Authentication → Settings → Authorized domains)

## 3. Create the Admin User (staff login)

### Step 1 — Create in Firebase Auth
1. **Authentication → Users → Add user**
2. Email: `kingfache@rental.com`
3. Password: `fache123`
4. Click **Add user**, then **copy the User UID**

### Step 2 — Add admin flag in Firestore
1. **Firestore Database → Data → Start collection** (or open `users`)
2. Collection ID: `users`
3. Document ID: *(paste the UID from step 1)*
4. Add fields:
   - `email` (string) = `kingfache@rental.com`
   - `isAdmin` (boolean) = `true`
5. **Save**

## 4. Cloudinary Setup (IMPORTANT — preset is missing!)

I verified via a test upload — **the upload preset `musk_chat_unsigned` does NOT exist on cloud `dfd1bgdam`**. This is why image uploads in the chat were failing with "Upload preset not found."

The chat widget now **falls back to Firebase Storage automatically** if Cloudinary fails — so images will work either way. But to use Cloudinary (which has better CDN performance and image optimization), create the preset:

1. Log into your **Cloudinary dashboard** (https://cloudinary.com/console)
2. Confirm the cloud name is `dfd1bgdam` (top-right of dashboard)
3. **Settings → Upload → Upload presets**
4. Click **"Add upload preset"**
5. Set:
   - **Name**: `musk_chat_unsigned`
   - **Signing Mode**: **Unsigned** ← critical
   - **Folder**: `rhc-chat` (optional)
6. Click **Save**

Now Cloudinary uploads will work. If you don't create the preset, images still upload successfully via Firebase Storage (same Firebase project, no extra setup).

## 5. Files

- `index.html` — public site (deploy to Vercel)
- `admin.html` — staff portal (deploy alongside)
- `vercel.json` — `/admin` → `/admin.html` rewrite
- `README.md` — quick-start guide

## 6. Deploy

1. Push all files to your GitHub repo (Akay-mighty/rental-home-connect-)
2. Vercel auto-redeploys in ~30s
3. Public site: `rental-home-connect.vercel.app`
4. Admin: `rental-home-connect.vercel.app/admin`
