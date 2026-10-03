# ElectraTrack — Firebase Setup Guide

## 1. Create a Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click **Add project** → give it a name (e.g., `electratrack`)
3. Disable Google Analytics (optional) → **Create project**

---

## 2. Enable Google Authentication

1. In the left sidebar: **Authentication** → **Get started**
2. Click **Sign-in method** tab
3. Click **Google** → toggle **Enable** → set your support email → **Save**
4. All other providers (email/password, anonymous, etc.) must remain **disabled**

---

## 3. Create Firestore Database

1. In the left sidebar: **Firestore Database** → **Create database**
2. Select **Start in production mode** → **Next**
3. Choose your preferred Cloud Firestore location → **Enable**

---

## 4. Deploy Firestore Security Rules

Option A — **Firebase Console** (quickest):
1. Firestore Database → **Rules** tab
2. Paste the contents of `firestore.rules` into the editor
3. Click **Publish**

Option B — **Firebase CLI**:
```bash
npm install -g firebase-tools
firebase login
firebase init firestore   # select your project
firebase deploy --only firestore:rules
```

The rules enforce **strict per-user isolation**:
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{document=**} {
      allow read, write: if request.auth != null
                         && request.auth.uid == userId;
    }
  }
}
```

---

## 5. Register a Web App & Get Config

1. In **Project Settings** (gear icon) → **Your apps** → click **</>** (Web)
2. Register app (give it a nickname, e.g., `electratrack-web`)
3. Copy the `firebaseConfig` object

---

## 6. Paste Config into index.html

Open `index.html` and replace the placeholder `FIREBASE_CONFIG` near the top of the `<script>` block:

```js
const FIREBASE_CONFIG = {
  apiKey:            "AIza...",          // ← your real values
  authDomain:        "your-project.firebaseapp.com",
  projectId:         "your-project",
  storageBucket:     "your-project.appspot.com",
  messagingSenderId: "1234567890",
  appId:             "1:1234567890:web:abcdef"
};
```

---

## 7. Authorise Your Domain

1. **Authentication** → **Settings** → **Authorised domains**
2. `localhost` is pre-authorised for development
3. Add your production domain (e.g., `electratrack.web.app`) when deploying

---

## 8. Deploy (Optional — Firebase Hosting)

```bash
firebase init hosting
# Public directory: . (root, since index.html is at root)
# Single-page app: Yes
# Overwrite index.html: No

firebase deploy --only hosting
```

Your app will be live at `https://YOUR_PROJECT_ID.web.app`

---

## Data Model Reference

```
Firestore
└── users/                          (collection)
    └── {uid}/                      (document — auto, per user)
        └── electricity_records/    (sub-collection)
            └── {recordId}/         (document — auto)
                ├── month     : number  (1–12)
                ├── year      : number  (e.g. 2024)
                ├── amount    : number  (₹, positive float)
                ├── units     : number  (kWh, positive float)
                └── timestamp : Timestamp (serverTimestamp)
```

---

## Security Checklist

- [x] Only `GoogleAuthProvider` is instantiated — no email/password or anonymous
- [x] `setPersistence(LOCAL)` keeps session alive across browser restarts
- [x] `onAuthStateChanged` gates all UI; DOM hidden until valid session confirmed
- [x] Sign-out destroys charts, nullifies all state arrays, calls `window.location.reload()`
- [x] Firestore rules deny all cross-user access at the database layer
- [x] All inputs validated client-side before any Firestore write
- [x] Duplicate month+year records blocked at form-submission time
- [x] `escapeHtml()` used on all user-data injected into `innerHTML`
