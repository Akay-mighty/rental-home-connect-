# Rental Home Connect — Admin Dashboard + Public Site

## What's in this package

| File | Purpose |
|------|---------|
| **index.html** | Public site (your existing React bundle + new chat widget + triple-click logo → admin) |
| **admin.html** | Staff portal — login, dashboard, live chat inbox, image manager, property CRUD, city manager, leads, settings |
| **FIRESTORE_SETUP.md** | Step-by-step Firebase setup (security rules, admin user creation, Cloudinary preset) |

## Quick start

### 1. Deploy the files
- Upload both `index.html` and `admin.html` to your Vercel project (or git push to https://github.com/Akay-mighty/rental-home-connect-)
- Public site: `https://rental-home-connect.vercel.app`
- Admin dashboard: `https://rental-home-connect.vercel.app/admin.html`

### 2. Set up Firebase (one-time)
Follow the steps in **`FIRESTORE_SETUP.md`** — it covers:
- Firestore security rules (paste-ready)
- Creating the admin user (`kingfache@rental.com` / `fache123`)
- Enabling anonymous auth (for visitor chat)
- Verifying your Cloudinary upload preset

### 3. Open the admin dashboard
- Visit `https://rental-home-connect.vercel.app/admin.html`
- Sign in with `kingfache@rental.com` / `fache123`
- Or: open the public site and **triple-click the logo** (within 2 seconds) — admin opens in a new tab

### 4. Test the live chat
- On the public site, tap the floating blue chat bubble (bottom-right)
- Enter your name and start chatting
- Open the admin dashboard → **Live Chat** — your message appears in real time
- Reply from admin → visitor sees the response instantly
- Both sides see typing indicators, support image attachments (via Cloudinary)

## Features

### Public site (`index.html`)
- Existing React bundle untouched
- **Triple-click logo** (within 2s) → opens admin dashboard
- **Floating chat widget** (bottom-right) with:
  - Name prompt for new visitors
  - Real-time messaging with admin
  - Image attachment upload (Cloudinary)
  - Typing indicators (both directions)
  - Persists chat across sessions (localStorage)
  - Anonymous auth auto-creates visitor uid
- **City carousels** section (auto-injected above listings when admin adds cities via dashboard)

### Admin dashboard (`admin.html`)
- **Dashboard** — KPIs (active chats, new leads, properties, cities, images)
- **Live Chat inbox** — real-time visitor conversations, typing indicators, image attachments, mark resolved
- **Leads** — contact form submissions + chat leads, mark resolved, export CSV
- **Properties** — full CRUD (add/edit/delete listings, upload images via Cloudinary)
- **Cities** — add/remove cities shown in homepage carousels
- **Image Manager** — bulk upload to Cloudinary, copy URL, delete records
- **Settings** — branding (logo, brand text, contact email/phone), social media (TikTok handles preset)

## Tech stack
- Firebase (existing `rental-home-connect` project) — Auth (anonymous + email/password), Firestore, Storage
- Cloudinary (reused creds from support wallet project: cloud `dfd1bgdam`, preset `musk_chat_unsigned`)
- Tailwind CSS (public site CDN), vanilla CSS (admin dashboard)
- Font Awesome 6.5.1 icons, Inter + Cormorant Garamond fonts

## Admin credentials (default)
- Email: `kingfache@rental.com`
- Password: `fache123`

You can change the password later in Firebase Console → Authentication.

## Pre-filled branding
- Contact email: `rentalhomeconnects@gmail.com`
- TikTok main: `_rentalhomeconnects`
- TikTok agents: `_christopherhayes`, `_ethanhayes1`
- Public site URL: `https://rental-home-connect.vercel.app`

## Support
If anything doesn't work, check:
1. Firebase console → Authentication → confirm `kingfache@rental.com` exists
2. Firestore → `users` collection → confirm the user's doc has `isAdmin: true`
3. Firestore rules match what's in `FIRESTORE_SETUP.md`
4. Cloudinary preset `musk_chat_unsigned` is set to **Unsigned**
