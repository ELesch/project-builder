# Firebase Provider Guide

Google's Backend-as-a-Service with real-time database, authentication, and hosting.

## When to Use

- Mobile-first applications
- Real-time collaboration features
- Google ecosystem integration
- When you need NoSQL database

## Setup Steps

1. **Create Project**
   - Go to console.firebase.google.com
   - Click "Add project" or select existing
   - Enable Google Analytics (optional)

2. **Add App**
   - Navigate to Project Settings > General > Your apps
   - Click "Add app" > Web (or appropriate platform)
   - Register app name

3. **Get Config**
   - Copy from the Firebase config object shown after adding app

## Environment Variables

```bash
# .env.local

# Public (safe for client-side)
NEXT_PUBLIC_FIREBASE_API_KEY=AIza...
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=project-id.firebaseapp.com
NEXT_PUBLIC_FIREBASE_PROJECT_ID=project-id
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=project-id.appspot.com
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=123456789
NEXT_PUBLIC_FIREBASE_APP_ID=1:123456789:web:abc123

# Server-side only (download from Project Settings > Service Accounts)
FIREBASE_ADMIN_SDK_KEY={"type":"service_account",...}
```

## Enable Services

### Firestore Database
- Build > Firestore Database > Create database
- Choose location and security rules

### Authentication
- Build > Authentication > Get started
- Enable sign-in providers (Email, Google, etc.)

### Storage
- Build > Storage > Get started
- Set security rules

## Client Setup (Next.js)

```typescript
// src/lib/firebase.ts
import { initializeApp, getApps } from 'firebase/app';
import { getAuth } from 'firebase/auth';
import { getFirestore } from 'firebase/firestore';

const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
  storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID,
};

const app = getApps().length === 0 ? initializeApp(firebaseConfig) : getApps()[0];

export const auth = getAuth(app);
export const db = getFirestore(app);
```

## Admin Setup (Server-side)

```typescript
// src/lib/firebase-admin.ts
import { initializeApp, cert, getApps } from 'firebase-admin/app';
import { getAuth } from 'firebase-admin/auth';
import { getFirestore } from 'firebase-admin/firestore';

const serviceAccount = JSON.parse(
  process.env.FIREBASE_ADMIN_SDK_KEY || '{}'
);

const app = getApps().length === 0
  ? initializeApp({ credential: cert(serviceAccount) })
  : getApps()[0];

export const adminAuth = getAuth(app);
export const adminDb = getFirestore(app);
```

## Verification

**IMPORTANT:** `firebase login` is interactive (opens browser for OAuth). Never run it from Claude Code.

```bash
# Install Firebase CLI
npm install -g firebase-tools

# Check authentication (will fail if not logged in)
firebase projects:list
# If NOT authenticated, ask user: "Please run `firebase login` in a separate terminal."

# After user confirms login, verify:
firebase projects:list

# Test locally
firebase emulators:start
```

## Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Permission denied | Security rules | Update Firestore/Storage rules |
| API key invalid | Wrong project | Check config matches project |
| Admin SDK error | Missing service account | Download from Project Settings |
| CORS error | Domain not added | Add domain to Authentication settings |

## Security Rules (Firestore)

```javascript
// firestore.rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

## Dependencies

```bash
# Client SDK
npm install firebase

# Admin SDK (server-side)
npm install firebase-admin
```
