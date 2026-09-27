# Rental Home Connect — Public Site + Admin Dashboard

## What's new in v3 (latest)

### Bug fixes
- ✅ **addDoc() PointerEvent bug fixed** — chat messages no longer try to save the click event to Firestore
- ✅ **Cloudinary "Upload preset not found" handled** — automatic fallback to Firebase Storage when preset is missing
- ✅ **Friendly error messages** — visitors see "Something went wrong, please try again" instead of raw Firebase/Cloudinary errors (real errors still logged to console)
- ✅ **Navbar clicks work** — nav links (Homes, Map Search, Neighborhoods, How It Works) now scroll to the right sections
- ✅ **Logo centered + clickable** — triple-click the logo to open admin, without blocking nav clicks

### Renter auth (NEW)
- Sign up / Sign in with email + password
- "Continue with Google" button
- Stored in `renters/{uid}` Firestore collection (separate from admin `users/{uid}`)
- "Saved Homes" page syncs to Firestore when signed in, falls back to localStorage when not
- Sign In button appears in the header next to "Start Application"

### New logos
- Public site header uses the new public logo (centered, 64px tall)
- Admin dashboard, login page, and public site footer use the admin logo

## Files

| File | Purpose |
|------|---------|
| **index.html** | Public site with chat widget, renter auth, nav fixes, new logos |
| **admin.html** | Staff portal with new admin logo |
| **vercel.json** | `/admin` → `/admin.html` rewrite |
| **FIRESTORE_SETUP.md** | Updated security rules + Cloudinary preset setup |
| **README.md** | This file |

## Quick start

1. **Read `FIRESTORE_SETUP.md`** — it covers:
   - Updated Firestore security rules (now includes `renters/{uid}` collection)
   - Enabling Email/Password + Google + Anonymous auth providers
   - Creating `kingfache@rental.com` admin user
   - Creating the missing Cloudinary upload preset `musk_chat_unsigned`

2. **Push to GitHub** → Vercel auto-deploys in ~30s

3. **Test**:
   - Public site: `rental-home-connect.vercel.app`
   - Admin: `rental-home-connect.vercel.app/admin`
   - Triple-click the logo to open admin from the public site
   - Click "Sign In" in the header to test renter auth

## Admin credentials (default)
- Email: `kingfache@rental.com`
- Password: `fache123`

## Tech stack
- Firebase (`rental-home-connect` project) — Auth (anonymous + email/password + Google), Firestore, Storage
- Cloudinary (cloud `dfd1bgdam`, preset `musk_chat_unsigned` — **must be created in dashboard**)
- Tailwind CSS (public site CDN), vanilla CSS (admin dashboard)
- Font Awesome 6.5.1 icons, Inter + Cormorant Garamond fonts

## Pre-filled branding
- Contact email: `rentalhomeconnects@gmail.com`
- TikTok main: `_rentalhomeconnects`
- TikTok agents: `_christopherhayes`, `_ethanhayes1`
- Public site URL: `https://rental-home-connect.vercel.app`
