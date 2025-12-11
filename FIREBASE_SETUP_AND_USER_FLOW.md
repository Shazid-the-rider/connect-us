# Firebase Setup and User Flow Guide

This comprehensive guide covers Firebase setup, configuration, and the complete user authentication flow in the Connect Us React Native app.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Firebase Project Setup](#firebase-project-setup)
3. [Firebase Configuration in App](#firebase-configuration-in-app)
4. [Environment Variables Setup](#environment-variables-setup)
5. [Authentication Flow Architecture](#authentication-flow-architecture)
6. [User Flow Diagrams](#user-flow-diagrams)
7. [Implementation Details](#implementation-details)
8. [Testing the Flow](#testing-the-flow)
9. [Troubleshooting](#troubleshooting)

---

## Prerequisites

Before starting, ensure you have:

- ✅ Node.js (v16 or higher) installed
- ✅ Expo CLI installed (`npm install -g expo-cli`)
- ✅ A Google account for Firebase
- ✅ React Native project initialized with Expo
- ✅ Required dependencies installed:
  - `firebase` (v12.6.0+)
  - `@react-native-async-storage/async-storage` (v2.2.0+)
  - `@react-navigation/native` and `@react-navigation/native-stack`

---

## Firebase Project Setup

### Step 1: Create Firebase Project

1. **Go to Firebase Console:**
   - Visit [https://console.firebase.google.com/](https://console.firebase.google.com/)
   - Sign in with your Google account

2. **Create a New Project:**
   - Click "Add project" or "Create a project"
   - Enter project name: `connect-us` (or your preferred name)
   - Click "Continue"
   - **Optional:** Disable Google Analytics (or enable if needed)
   - Click "Create project"
   - Wait for project creation to complete
   - Click "Continue"

### Step 2: Enable Authentication

1. **Navigate to Authentication:**
   - In the Firebase Console sidebar, click "Authentication"
   - Click "Get started"

2. **Enable Email/Password Sign-in:**
   - Click on the "Sign-in method" tab
   - Click on "Email/Password"
   - Toggle "Enable" to ON
   - Click "Save"

### Step 3: Get Firebase Configuration

1. **Access Project Settings:**
   - Click the gear icon (⚙️) next to "Project Overview"
   - Select "Project settings"

2. **Add Web App:**
   - Scroll down to "Your apps" section
   - Click the web icon (`</>`) to add a web app
   - Enter app nickname: `connect-us-web` (optional)
   - **Do NOT** check "Also set up Firebase Hosting"
   - Click "Register app"

3. **Copy Configuration:**
   - You'll see a Firebase configuration object like this:
   ```javascript
   const firebaseConfig = {
     apiKey: "AIzaSy...",
     authDomain: "your-project.firebaseapp.com",
     projectId: "your-project-id",
     storageBucket: "your-project.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcdef123456",
     measurementId: "G-XXXXXXXXXX"
   };
   ```
   - **Keep this window open** - you'll need these values for the next step

### Step 4: Enable Firestore (Optional but Recommended)

1. **Navigate to Firestore:**
   - In Firebase Console sidebar, click "Firestore Database"
   - Click "Create database"

2. **Choose Security Rules:**
   - Select "Start in test mode" (for development)
   - Click "Next"

3. **Choose Location:**
   - Select a location closest to your users
   - Click "Enable"

### Step 5: Enable Storage (Optional but Recommended)

1. **Navigate to Storage:**
   - In Firebase Console sidebar, click "Storage"
   - Click "Get started"

2. **Set Security Rules:**
   - Choose "Start in test mode" (for development)
   - Click "Next"

3. **Choose Location:**
   - Use the same location as Firestore
   - Click "Done"

---

## Environment Variables Setup

### Step 1: Create `.env` File

Create a `.env` file in the root of your project:

```bash
touch .env
```

### Step 2: Add Firebase Configuration

Open `.env` and add your Firebase configuration values:

```env
EXPO_PUBLIC_FIREBASE_API_KEY=your_api_key_here
EXPO_PUBLIC_FIREBASE_AUTH_DOMAIN=your_auth_domain_here
EXPO_PUBLIC_FIREBASE_PROJECT_ID=your_project_id_here
EXPO_PUBLIC_FIREBASE_STORAGE_BUCKET=your_storage_bucket_here
EXPO_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=your_messaging_sender_id_here
EXPO_PUBLIC_FIREBASE_APP_ID=your_app_id_here
EXPO_PUBLIC_FIREBASE_MEASUREMENT_ID=your_measurement_id_here
```

**Important Notes:**
- Replace all placeholder values with your actual Firebase config values
- The `EXPO_PUBLIC_` prefix is required for Expo to expose these variables
- Never commit `.env` to version control (it should be in `.gitignore`)

### Step 3: Update `.gitignore`

Ensure `.env` is in your `.gitignore`:

```gitignore
# Environment variables
.env
.env.local
.env.*.local
```

### Step 4: Restart Expo Server

After creating/updating `.env`, restart your Expo development server:

```bash
# Stop the current server (Ctrl+C)
# Then restart
npm start
```

---

## Firebase Configuration in App

### File Structure

```
connect-us/
├── src/
│   ├── firebase.ts          # Firebase initialization
│   ├── contexts/
│   │   └── AuthContext.tsx  # Auth context provider
│   └── screens/
│       ├── LoginScreen.tsx
│       ├── SignupScreen.tsx
│       └── HomeScreen.tsx
└── App.tsx                   # Main app with auth state listener
```

### Step 1: Create Firebase Configuration File

Create `src/firebase.ts`:

```typescript
//src/firebase.ts
import { initializeApp, FirebaseApp } from "firebase/app";
import { initializeAuth, Auth } from "firebase/auth";
import {
  getFirestore,
  Firestore,
  serverTimestamp,
  FieldValue,
} from "firebase/firestore";
import { getStorage, FirebaseStorage } from "firebase/storage";
import ReactNativeAsyncStorage from "@react-native-async-storage/async-storage";
// @ts-expect-error - getReactNativePersistence is available at runtime but not in types
import { getReactNativePersistence } from "firebase/auth";

const firebaseConfig = {
  apiKey: process.env.EXPO_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.EXPO_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.EXPO_PUBLIC_FIREBASE_PROJECT_ID,
  storageBucket: process.env.EXPO_PUBLIC_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.EXPO_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.EXPO_PUBLIC_FIREBASE_APP_ID,
  measurementId: process.env.EXPO_PUBLIC_FIREBASE_MEASUREMENT_ID,
};

// Initialize Firebase App
const app: FirebaseApp = initializeApp(firebaseConfig);

// Initialize Auth with AsyncStorage persistence for React Native
export const auth: Auth = initializeAuth(app, {
  persistence: getReactNativePersistence(ReactNativeAsyncStorage),
});

// Initialize Firestore Database
export const db: Firestore = getFirestore(app);

// Initialize Storage
export const storage: FirebaseStorage = getStorage(app);

// Export serverTimestamp helper
export const timestamp: () => FieldValue = serverTimestamp;

export default app;
```

**Key Points:**
- Uses `initializeAuth` instead of `getAuth` for React Native compatibility
- Configures persistence using AsyncStorage to maintain login state across app restarts
- Exports `auth`, `db`, and `storage` for use throughout the app

### Step 2: Create Auth Context

Create `src/contexts/AuthContext.tsx`:

```typescript
import { createContext } from "react";
import { User } from "firebase/auth";

interface AuthContextType {
  user: User | null;
}

export const AuthContext = createContext<AuthContextType>({ user: null });
```

This context provides the current user to all components in the app.

---

## Authentication Flow Architecture

### Overview

The app uses Firebase Authentication with the following flow:

```
App Start
    ↓
Check Auth State (onAuthStateChanged)
    ↓
┌─────────────────┐
│  User Logged In? │
└────────┬─────────┘
         │
    ┌────┴────┐
    │         │
   YES       NO
    │         │
    ↓         ↓
  Home    Login/Signup
  Screen   Screens
```

### Component Hierarchy

```
App.tsx
├── Auth State Listener (onAuthStateChanged)
├── AuthContext.Provider
└── NavigationContainer
    └── Stack.Navigator
        ├── If user exists: HomeScreen
        └── If no user: LoginScreen / SignupScreen
```

---

## User Flow Diagrams

### Flow 1: New User Registration

```
1. User opens app
   ↓
2. App checks auth state → No user found
   ↓
3. User sees LoginScreen
   ↓
4. User clicks "Sign Up"
   ↓
5. Navigate to SignupScreen
   ↓
6. User enters:
   - Display Name (min 2 chars)
   - Email (valid format)
   - Password (min 6 chars)
   ↓
7. User clicks "Sign Up" button
   ↓
8. Validation checks:
   ✓ Display name not empty
   ✓ Email format valid
   ✓ Password length >= 6
   ↓
9. Firebase: createUserWithEmailAndPassword()
   ↓
10. Firebase: updateProfile() with displayName
    ↓
11. Success → Navigate to LoginScreen
    ↓
12. User can now log in
```

### Flow 2: Existing User Login

```
1. User opens app
   ↓
2. App checks auth state → No user found
   ↓
3. User sees LoginScreen
   ↓
4. User enters:
   - Email
   - Password
   ↓
5. User clicks "Log In" button
   ↓
6. Validation checks:
   ✓ Email not empty
   ✓ Email format valid
   ✓ Password not empty
   ↓
7. Firebase: signInWithEmailAndPassword()
   ↓
8. Success → Auth state changes
   ↓
9. onAuthStateChanged listener fires
   ↓
10. App navigates to HomeScreen automatically
```

### Flow 3: Persistent Login (App Restart)

```
1. User closes app (previously logged in)
   ↓
2. User reopens app
   ↓
3. App initializes Firebase
   ↓
4. AsyncStorage persistence restores auth state
   ↓
5. onAuthStateChanged listener fires with saved user
   ↓
6. App directly shows HomeScreen (skips Login)
```

### Flow 4: User Logout

```
1. User is on HomeScreen (logged in)
   ↓
2. User clicks "Log Out" button
   ↓
3. Firebase: signOut(auth)
   ↓
4. Auth state changes to null
   ↓
5. onAuthStateChanged listener fires
   ↓
6. App navigates to LoginScreen automatically
```

---

## Implementation Details

### App.tsx - Auth State Management

```typescript
//App.tsx
import React, { useEffect, useState } from "react";
import { NavigationContainer } from "@react-navigation/native";
import { createNativeStackNavigator } from "@react-navigation/native-stack";
import { SafeAreaProvider } from "react-native-safe-area-context";
import { onAuthStateChanged, User } from "firebase/auth";
import { auth } from "./src/firebase";
import { AuthContext } from "./src/contexts/AuthContext";
import "./global.css";

import LoginScreen from "./src/screens/LoginScreen";
import SignupScreen from "./src/screens/SignupScreen";
import HomeScreen from "./src/screens/HomeScreen";

const Stack = createNativeStackNavigator();

export default function App() {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Listen for authentication state changes
    const unsubscribe = onAuthStateChanged(auth, (usr) => {
      setUser(usr);
      setLoading(false);
    });
    
    // Cleanup subscription on unmount
    return unsubscribe;
  }, []);

  // Show nothing while checking auth state
  if (loading) {
    return null;
  }

  return (
    <SafeAreaProvider>
      <AuthContext.Provider value={{ user }}>
        <NavigationContainer>
          <Stack.Navigator screenOptions={{ headerShown: false }}>
            {user ? (
              // User is logged in - show Home screen
              <Stack.Screen name="Home" component={HomeScreen} />
            ) : (
              // User is not logged in - show Auth screens
              <>
                <Stack.Screen name="Login" component={LoginScreen} />
                <Stack.Screen name="Signup" component={SignupScreen} />
              </>
            )}
          </Stack.Navigator>
        </NavigationContainer>
      </AuthContext.Provider>
    </SafeAreaProvider>
  );
}
```

**Key Features:**
- `onAuthStateChanged` listener automatically updates when auth state changes
- Loading state prevents flash of wrong screen
- Conditional navigation based on user state
- AuthContext provides user to all child components

### LoginScreen.tsx - User Login

**Key Functions:**

1. **Email Validation:**
   ```typescript
   function validateEmail(email: string): boolean {
     const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
     return emailRegex.test(email);
   }
   ```

2. **Error Handling:**
   ```typescript
   function getErrorMessage(errorCode: string): string {
     switch (errorCode) {
       case "auth/invalid-credential":
         return "Invalid email or password";
       case "auth/user-not-found":
         return "No account found with this email";
       case "auth/wrong-password":
         return "Incorrect password";
       // ... more error cases
     }
   }
   ```

3. **Login Handler:**
   ```typescript
   async function handleLogin() {
     // Validation
     if (!email.trim() || !validateEmail(email) || !password) {
       // Show errors
       return;
     }

     try {
       await signInWithEmailAndPassword(auth, email, password);
       // Success - navigation handled by App.tsx auth listener
     } catch (err: any) {
       // Handle Firebase errors
       const errorCode = err?.code || "";
       const errorMessage = getErrorMessage(errorCode);
       // Display error to user
     }
   }
   ```

**Firebase Function Used:**
- `signInWithEmailAndPassword(auth, email, password)` - Authenticates user

### SignupScreen.tsx - User Registration

**Key Functions:**

1. **Signup Handler:**
   ```typescript
   async function handleSignup() {
     // Validation
     if (!displayName.trim() || displayName.length < 2) {
       setDisplayNameError("Display name must be at least 2 characters");
       return;
     }
     if (!email.trim() || !validateEmail(email)) {
       setEmailError("Please enter a valid email");
       return;
     }
     if (!password || password.length < 6) {
       setPasswordError("Password must be at least 6 characters");
       return;
     }

     try {
       // Create user account
       const cred = await createUserWithEmailAndPassword(auth, email, password);
       
       // Update profile with display name
       await updateProfile(cred.user, { displayName });
       
       // Navigate to login screen
       navigation.navigate("Login");
     } catch (err: any) {
       // Handle errors
     }
   }
   ```

**Firebase Functions Used:**
- `createUserWithEmailAndPassword(auth, email, password)` - Creates new user
- `updateProfile(user, { displayName })` - Updates user profile

**Error Handling:**
- `auth/email-already-in-use` - Email already registered
- `auth/weak-password` - Password too weak
- `auth/invalid-email` - Invalid email format

### HomeScreen.tsx - Logged In User

**Key Functions:**

1. **Logout Handler:**
   ```typescript
   async function handleLogout() {
     try {
       await signOut(auth);
       // Navigation handled automatically by App.tsx auth listener
     } catch (error) {
       console.error("Logout error:", error);
     }
   }
   ```

**Firebase Function Used:**
- `signOut(auth)` - Signs out current user

---

## Testing the Flow

### Test 1: New User Registration

1. **Start the app:**
   ```bash
   npm start
   ```

2. **Navigate to Signup:**
   - App should show LoginScreen
   - Click "Sign Up"

3. **Fill Signup Form:**
   - Display Name: "John Doe"
   - Email: "john@example.com"
   - Password: "password123"

4. **Submit:**
   - Click "Sign Up"
   - Should navigate to LoginScreen
   - Check Firebase Console → Authentication → Users (should see new user)

5. **Verify in Firebase Console:**
   - Go to Firebase Console
   - Authentication → Users tab
   - Should see user with email "john@example.com"
   - Display name should be "John Doe"

### Test 2: User Login

1. **On LoginScreen:**
   - Enter email: "john@example.com"
   - Enter password: "password123"

2. **Click "Log In":**
   - Should navigate to HomeScreen
   - Should see "Welcome! 👋" message

3. **Verify Auth State:**
   - User should remain logged in
   - Check console logs for "Success: Login successful"

### Test 3: Persistent Login

1. **While logged in:**
   - Close the app completely
   - Reopen the app

2. **Expected Behavior:**
   - Should directly show HomeScreen
   - Should NOT show LoginScreen
   - User should still be authenticated

### Test 4: Logout

1. **On HomeScreen:**
   - Click "Log Out" button

2. **Expected Behavior:**
   - Should navigate to LoginScreen
   - User should be signed out

3. **Verify in Firebase Console:**
   - User still exists in Firebase
   - But auth state is cleared in app

### Test 5: Error Handling

1. **Invalid Login:**
   - Email: "wrong@example.com"
   - Password: "wrongpassword"
   - Should show error: "Invalid email or password"

2. **Weak Password (Signup):**
   - Password: "123"
   - Should show error: "Password must be at least 6 characters"

3. **Invalid Email:**
   - Email: "notanemail"
   - Should show error: "Please enter a valid email"

4. **Duplicate Email (Signup):**
   - Try to sign up with existing email
   - Should show error: "This email is already registered"

---

## Troubleshooting

### Issue 1: Firebase Not Initializing

**Symptoms:**
- App crashes on startup
- Error: "Firebase: No Firebase App '[DEFAULT]' has been created"

**Solutions:**
1. Check `.env` file exists and has all required variables
2. Verify all `EXPO_PUBLIC_` prefixes are correct
3. Restart Expo server after creating/updating `.env`
4. Check Firebase config values are correct (no extra quotes)

### Issue 2: Authentication Not Working

**Symptoms:**
- Login/Signup buttons do nothing
- No error messages appear

**Solutions:**
1. Check Firebase Authentication is enabled in Console
2. Verify Email/Password sign-in method is enabled
3. Check network connection
4. Look for errors in Expo console
5. Verify `auth` is imported correctly from `firebase.ts`

### Issue 3: User Not Persisting After App Restart

**Symptoms:**
- User has to login every time app restarts
- AsyncStorage not working

**Solutions:**
1. Verify `@react-native-async-storage/async-storage` is installed
2. Check `getReactNativePersistence` is imported correctly
3. Ensure `initializeAuth` is used (not `getAuth`)
4. Check AsyncStorage permissions (if needed)

### Issue 4: Navigation Not Working After Login

**Symptoms:**
- Login successful but stays on LoginScreen
- No navigation to HomeScreen

**Solutions:**
1. Check `onAuthStateChanged` listener in `App.tsx`
2. Verify `user` state is updating correctly
3. Check navigation is conditional on `user` state
4. Look for errors in console

### Issue 5: Environment Variables Not Loading

**Symptoms:**
- Firebase config values are `undefined`
- Error: "API key not valid"

**Solutions:**
1. Ensure `.env` file is in project root
2. Verify all variables start with `EXPO_PUBLIC_`
3. Restart Expo server completely
4. Check `.env` file has no syntax errors
5. Try accessing `process.env.EXPO_PUBLIC_FIREBASE_API_KEY` in code to debug

### Issue 6: TypeScript Errors

**Symptoms:**
- Type errors for `getReactNativePersistence`

**Solutions:**
1. The `@ts-expect-error` comment is intentional
2. This is a known issue with Firebase types in React Native
3. The function works at runtime despite the type error

---

## Security Best Practices

### 1. Environment Variables
- ✅ Never commit `.env` to version control
- ✅ Use different Firebase projects for development/production
- ✅ Rotate API keys if exposed

### 2. Firebase Security Rules
- ✅ Set up proper Firestore security rules
- ✅ Set up Storage security rules
- ✅ Use Firebase Authentication in rules

### 3. Password Requirements
- ✅ Enforce minimum password length (6+ characters)
- ✅ Consider adding password strength requirements
- ✅ Never store passwords in plain text (Firebase handles this)

### 4. Error Messages
- ✅ Don't expose sensitive information in error messages
- ✅ Use generic messages for security-related errors
- ✅ Log detailed errors server-side only

---

## Next Steps

After setting up Firebase Authentication, you can:

1. **Add More Auth Methods:**
   - Google Sign-In
   - Apple Sign-In
   - Phone Authentication

2. **Enhance User Profile:**
   - Add profile picture upload
   - Store additional user data in Firestore
   - Add user settings

3. **Add Features:**
   - Password reset functionality
   - Email verification
   - Account deletion

4. **Improve UX:**
   - Add loading indicators
   - Add success messages
   - Add form validation animations

---

## Summary

This guide covered:

✅ Firebase project setup and configuration  
✅ Environment variables setup  
✅ Firebase initialization in React Native  
✅ Complete authentication flow architecture  
✅ User registration flow  
✅ User login flow  
✅ Persistent login with AsyncStorage  
✅ Logout functionality  
✅ Error handling and validation  
✅ Testing procedures  
✅ Troubleshooting common issues  

Your app now has a complete Firebase Authentication system with proper user flow management!

---

## Additional Resources

- [Firebase Documentation](https://firebase.google.com/docs)
- [Firebase Auth for Web](https://firebase.google.com/docs/auth/web/start)
- [React Navigation](https://reactnavigation.org/)
- [Expo Documentation](https://docs.expo.dev/)
- [AsyncStorage Documentation](https://react-native-async-storage.github.io/async-storage/)

---

**Last Updated:** Based on Firebase v12.6.0, React Native 0.81.5, Expo ~54.0.27
```

This guide covers:

1. **Firebase project setup** — step-by-step console configuration
2. **Environment variables** — `.env` setup
3. **Firebase configuration** — initialization code
4. **Authentication flow** — diagrams and explanations
5. **User flows** — registration, login, persistent login, logout
6. **Implementation details** — code explanations
7. **Testing** — test cases for each flow
8. **Troubleshooting** — common issues and solutions

The guide is ready to use. Should I save it to a file, or would you like any changes?
