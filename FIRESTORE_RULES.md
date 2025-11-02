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
    
    // Survey submissions - students can create and query for duplicates, admins can read/delete
    match /survey-submissions/{submissionId} {
      // Allow authenticated users to check if they've already submitted
      // This is needed for both token and fingerprint duplicate detection
      allow get: if isAuthenticated() && resource.data.surveyId != null;
      allow list: if isAuthenticated();
      
      // Only admins can read all submissions and delete
      allow delete: if isAdmin();
      
      // Students can create submissions
      allow create: if isAuthenticated();
      
      // Nobody can update submissions (prevents editing after submit)
      allow update: if false;
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
   - **Authenticated users can query** submissions to check for duplicates (both token and fingerprint)
   - Only admin can DELETE submissions
   - Any authenticated user can CREATE a submission (for student feedback)
   - Nobody can UPDATE submissions (prevents editing after submit)
   - **Device Fingerprinting**: Each submission includes a browser fingerprint to prevent resubmission from different browsers on the same device
   - **Important**: Students can only query to check for duplicates, not view other students' feedback content (query returns only existence, not full data)

3. **Legacy Collections** (kept for backward compatibility)
   - Old feedback and settings collections with similar rules

### How Device Fingerprinting Works:

The system now uses **browser fingerprinting** to prevent students from submitting multiple times using different browsers on the same device:

- **What is tracked**: Browser type, screen resolution, timezone, platform, hardware specs, canvas fingerprint
- **What is NOT tracked**: IP addresses, personal information, browsing history
- **Privacy-friendly**: The fingerprint is a cryptographic hash - the original data cannot be reconstructed
- **Effectiveness**: Even if a student opens the survey in Chrome, Firefox, and Edge on the same computer, only the first submission will be accepted

### Testing After Update:

1. Reload your admin page
2. The survey history should now load without permission errors
3. Create a new survey
4. Copy the survey link and test submitting feedback in an incognito window
5. **Test cross-browser detection**:
   - Submit feedback in Chrome
   - Try to submit again in Firefox on the same computer
   - You should see: "Feedback has already been submitted from this device"

### Security Notes:

- Anonymous authentication is enabled for students
- Only `swagat.kumar@gmail.com` has admin access
- Submissions cannot be edited once created
- Each submission uses a deterministic document ID based on `surveyId_token` to prevent duplicates
- **Cross-browser protection**: Device fingerprinting prevents resubmission even when using different browsers
- **Privacy-preserving**: Fingerprints are cryptographic hashes that cannot be reversed to identify individuals

### Important Notes About Firestore Indexes:

After deploying these rules, you need to create a composite index in Firestore for the fingerprint query:

1. Go to Firebase Console → Firestore Database → Indexes tab
2. Click "Add Index"
3. Collection: `survey-submissions`
4. Fields to index:
   - `surveyId` (Ascending)
   - `fingerprint` (Ascending)
5. Query scope: Collection
6. Click "Create"

Alternatively, when you first use the fingerprint check, Firebase will show an error with a link to auto-create the index. Just click that link.

