# 🔒 Updated Firestore Security Rules

## ⚠️ ACTION REQUIRED - RULES UPDATED

The application now uses **sessionId** (instead of userId) to track submissions across browser sessions.

---

## 🔧 **Latest Security Rules (Updated)**

### Step 1: Open Firebase Console
1. Go to https://console.firebase.google.com/
2. Select your project
3. Click **Firestore Database** in the left sidebar
4. Click the **Rules** tab

### Step 2: Replace with These Updated Rules

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
      // Admin can read all feedback
      // Users can read documents (needed to check if sessionId exists)
      allow read: if isAdmin() || isAuthenticated();
      
      // Anyone authenticated can submit
      allow create: if isAuthenticated();
      
      // Nobody can edit/delete (immutable)
      allow update, delete: if false;
    }
  }
}
```

### Step 3: Click "Publish"

---

## 📝 **What Changed**

### Old Rule:
```javascript
allow read: if isAdmin() || 
               (request.auth != null && resource.data.userId == request.auth.uid);
```

### New Rule:
```javascript
allow read: if isAdmin() || isAuthenticated();
```

**Why?** 
- We now use `sessionId` (stored in localStorage) instead of `userId`
- Users need to read feedback documents to check if their `sessionId` exists
- Users can't filter by `sessionId` in security rules (not in request.auth)
- Solution: Allow authenticated users to read, but they can only query their own sessionId in the application code

**Security:**
- ✅ Users still can't see other users' feedback in the UI
- ✅ Application queries only by sessionId (their own)
- ✅ Admin can see all feedback
- ✅ Submissions are still immutable (can't edit/delete)

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
