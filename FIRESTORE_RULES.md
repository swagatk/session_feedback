# Firestore Security Rules

## Critical: Update Your Firestore Rules

The error "Missing or insufficient permissions" means your Firestore security rules need to be updated.

### How to Update Firestore Rules:

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Select your project: `module-8192d`
3. Click on **Firestore Database** in the left menu
4. Click on the **Rules** tab
5. Replace the existing rules with the rules below
6. Click **Publish**

### Required Firestore Security Rules:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Helper function to check if user is authenticated admin
    function isAdmin() {
      return request.auth != null 
        && request.auth.token.email != null
        && request.auth.token.email == 'swagat.kumar@gmail.com';
    }
    
    // Helper function to check if user is authenticated (anonymous or email)
    function isAuthenticated() {
      return request.auth != null;
    }
    
    // Surveys collection - admins can read/write, students can read active surveys
    match /surveys/{surveyId} {
      allow read: if isAuthenticated();
      allow create, update, delete: if isAdmin();
    }
    
    // Survey submissions - students can create (once per token), admins can read/delete
    match /survey-submissions/{submissionId} {
      allow read, delete: if isAdmin();
      allow create: if isAuthenticated();
      allow update: if false; // Prevent editing after submission
    }
    
    // Legacy collections (for backward compatibility)
    match /lecture-feedback/{feedbackId} {
      allow read, delete: if isAdmin();
      allow create: if isAuthenticated();
    }
    
    match /app-settings/{document=**} {
      allow read: if isAuthenticated();
      allow write: if isAdmin();
    }
    
    match /admin/{document=**} {
      allow read, write: if isAdmin();
    }
  }
}
```

### What These Rules Do:

1. **Surveys Collection** (`/surveys/{surveyId}`)
   - Anyone authenticated (including anonymous users) can READ surveys
   - Only the admin email can CREATE, UPDATE, or DELETE surveys

2. **Survey Submissions** (`/survey-submissions/{submissionId}`)
   - Only admin can READ and DELETE submissions
   - Any authenticated user can CREATE a submission (for student feedback)
   - Nobody can UPDATE submissions (prevents editing after submit)

3. **Legacy Collections** (kept for backward compatibility)
   - Old feedback and settings collections with similar rules

### Testing After Update:

1. Reload your admin page
2. The survey history should now load without permission errors
3. Create a new survey
4. Copy the survey link and test submitting feedback in an incognito window

### Security Notes:

- Anonymous authentication is enabled for students
- Only `swagat.kumar@gmail.com` has admin access
- Submissions cannot be edited once created
- Each submission uses a deterministic document ID based on `surveyId_token` to prevent duplicates
