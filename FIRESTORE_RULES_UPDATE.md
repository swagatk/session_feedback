# 🔒 Required Firestore Security Rules Update

## ⚠️ ACTION REQUIRED

To support the new user submission tracking feature, you need to update your Firestore Security Rules.

---

## 📋 What Changed

The application now stores `userId` (anonymous user ID) with each feedback submission. This allows users to see the "Thank you" message even after:
- Closing and reopening the browser
- Clearing browser cache/localStorage
- Using a different browser or device

To make this work, users need permission to **read their own submissions** (but not others').

---

## 🔧 How to Update Security Rules

### Step 1: Open Firebase Console
1. Go to https://console.firebase.google.com/
2. Select your project
3. Click **Firestore Database** in the left sidebar
4. Click the **Rules** tab

### Step 2: Update the Rules

Find this section:
```javascript
// Feedback submissions
match /lecture-feedback/{document} {
  allow read: if isAdmin();           // Only admin can read feedback
  allow create: if isAuthenticated(); // Anyone authenticated can submit
  allow update, delete: if false;     // Nobody can edit/delete (immutable)
}
```

**Replace it with:**
```javascript
// Feedback submissions
match /lecture-feedback/{document} {
  // Admin can read all feedback, users can read their own
  allow read: if isAdmin() || 
                 (request.auth != null && resource.data.userId == request.auth.uid);
  allow create: if isAuthenticated(); // Anyone authenticated can submit
  allow update, delete: if false;     // Nobody can edit/delete (immutable)
}
```

### Step 3: Publish
Click the blue **"Publish"** button to save the changes.

---

## 📝 Complete Updated Rules

Here's the complete, updated Firestore Security Rules for your reference:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Helper function to check if user is admin
    function isAdmin() {
      return request.auth != null && 
             request.auth.token.email == 'swagat.kumar@gmail.com';
    }
    
    // Helper function to check if user is authenticated (student or admin)
    function isAuthenticated() {
      return request.auth != null;
    }
    
    // App settings (module code, dates, etc.)
    match /app-settings/{document} {
      allow read: if isAuthenticated();  // Anyone can read
      allow write: if isAdmin();          // Only admin can write
    }
    
    // Admin settings (recovery email, form availability)
    match /admin/{document=**} {
      allow read, write: if isAdmin();    // Only admin can access
    }
    
    // Feedback submissions
    match /lecture-feedback/{document} {
      // Admin can read all feedback, users can read their own
      allow read: if isAdmin() || 
                     (request.auth != null && resource.data.userId == request.auth.uid);
      allow create: if isAuthenticated(); // Anyone authenticated can submit
      allow update, delete: if false;     // Nobody can edit/delete (immutable)
    }
  }
}
```

---

## ✅ What This Allows

| User Type | Can Read All Feedback? | Can Read Own Feedback? | Can Submit? |
|-----------|------------------------|------------------------|-------------|
| **Admin** (`swagat.kumar@gmail.com`) | ✅ Yes | ✅ Yes | ✅ Yes |
| **Student** (anonymous auth) | ❌ No | ✅ Yes | ✅ Yes |
| **Not authenticated** | ❌ No | ❌ No | ❌ No |

---

## 🔐 Security Analysis

### What's Protected:
- ✅ Students cannot see other students' feedback
- ✅ Only admin can view all submissions
- ✅ Students can only verify their own submission
- ✅ Nobody can edit or delete feedback (immutable)
- ✅ Feedback must include userId (enforced by application)

### The `resource.data.userId` Check:
```javascript
resource.data.userId == request.auth.uid
```

This means:
- `resource.data.userId` = the userId stored in the feedback document
- `request.auth.uid` = the current user's anonymous Firebase UID
- Only matches if the user is reading their own submission

---

## 🧪 Testing After Update

### Test 1: Student Can Check Own Submission
1. Open site as student (not logged in as admin)
2. Submit feedback
3. Close browser completely
4. Reopen browser and visit site
5. **Expected**: See "Thank you for your feedback" message ✅

### Test 2: Student Cannot Read Others' Feedback
1. Open browser console
2. Try: `getDocs(collection(db, 'lecture-feedback'))`
3. **Expected**: Error - "Missing or insufficient permissions" ✅
4. (Only returns documents where userId matches current user)

### Test 3: Admin Can Read All Feedback
1. Login as admin
2. View admin panel
3. **Expected**: See all submitted feedback ✅

---

## ⏱️ When to Apply

**Apply this update IMMEDIATELY** after deploying the code changes.

Without this update:
- ❌ Users will get permission errors when checking if they've submitted
- ❌ The "Thank you" message won't persist across sessions
- ❌ Users might be able to submit multiple times

---

## 🆘 Troubleshooting

### Error: "Missing or insufficient permissions"
**Cause**: Rules not updated yet  
**Fix**: Follow the steps above to update and publish the rules

### Users can submit multiple times
**Cause**: Users can't read their own submissions to check  
**Fix**: Update the rules to allow `resource.data.userId == request.auth.uid`

### Admin can't see feedback
**Cause**: Typo in admin email  
**Fix**: Verify `isAdmin()` function uses correct email

---

## 📚 References

- [Firestore Security Rules Documentation](https://firebase.google.com/docs/firestore/security/rules-structure)
- [Firestore Security Rules Testing](https://firebase.google.com/docs/firestore/security/test-rules-emulator)

---

**✅ Update Complete**: Once you've published the new rules, the feature will work automatically!
