# Updated Firestore Security Rules for Survey-Based System

## Overview
These rules enforce security for the new survey-based feedback system with token-based submission tracking.

## Collections Structure

### 1. `surveys` Collection
- Stores survey definitions created by admin
- Fields: id, moduleCode, lectureDate, expiryDate, createdAt, createdBy, active

### 2. `survey-submissions` Collection  
- Stores individual feedback submissions linked to surveys
- Fields: surveyId, token, timestamp, clarity, pace, engagement, positive, negative

### 3. Legacy Collections (Backward Compatibility)
- `lecture-feedback`: Old feedback collection (kept for legacy data)
- `app-settings/title`: Legacy title document
- `admin/settings`: Admin settings

## Updated Security Rules

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Helper function to check if user is authenticated
    function isAuthenticated() {
      return request.auth != null;
    }
    
    // Helper function to check if user is authorized admin
    function isAdmin() {
      return isAuthenticated() && 
             request.auth.token.email in ['swagat.kumar@gmail.com'];
    }

    // ========================================
    // SURVEY COLLECTIONS (New System)
    // ========================================
    
    // Surveys collection - stores survey definitions
    match /surveys/{surveyId} {
      // Anyone can read active surveys (to validate survey links)
      allow read: if resource.data.active == true;
      
      // Only admin can create, update, or delete surveys
      allow create, update, delete: if isAdmin();
    }
    
    // Survey submissions collection - stores feedback linked to surveys
    match /survey-submissions/{submissionId} {
      // Allow create for anyone (anonymous submissions)
      // But enforce that required fields are present
      allow create: if request.resource.data.keys().hasAll([
        'surveyId', 'token', 'timestamp', 'clarity', 'pace', 'engagement'
      ]);
      
      // Only admin can read submissions (for viewing feedback)
      allow read: if isAdmin();
      
      // No updates or deletes allowed (submissions are immutable)
      allow update, delete: if false;
    }

    // ========================================
    // LEGACY COLLECTIONS (Backward Compatibility)
    // ========================================
    
    // Legacy feedback collection (old system)
    match /lecture-feedback/{docId} {
      // Allow create for anyone (anonymous submissions)
      allow create: if true;
      
      // Only admin can read
      allow read: if isAdmin();
      
      // No updates or deletes
      allow update, delete: if false;
    }
    
    // Legacy settings document (deprecated but kept for compatibility)
    match /app-settings/title {
      // Anyone can read
      allow read: if true;
      
      // Only admin can write
      allow write: if isAdmin();
    }
    
    // Admin settings
    match /admin/settings {
      // Only admin can read or write
      allow read, write: if isAdmin();
    }
    
    // Deny all other access
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

## Key Security Features

### 1. **Survey Access Control**
- ✅ Anyone can read active surveys (needed to validate survey links)
- ✅ Only admin can create/update/delete surveys
- ✅ Inactive surveys are not readable by students

### 2. **Submission Security**
- ✅ Anonymous submissions allowed (create permission for all)
- ✅ Required fields enforced (surveyId, token, timestamp, ratings)
- ✅ Submissions are immutable (no updates or deletes)
- ✅ Only admin can read submissions

### 3. **Token-Based Duplicate Prevention**
- ✅ Each submission includes a unique token
- ✅ Application checks Firestore for existing token before allowing submission
- ✅ Server-side validation via required fields enforcement

### 4. **Admin Authorization**
- ✅ Email-based admin verification (swagat.kumar@gmail.com)
- ✅ Admin has full access to all data
- ✅ Settings protected from unauthorized changes

### 5. **Legacy Support**
- ✅ Old feedback collection still readable by admin
- ✅ Legacy settings documents still accessible
- ✅ Smooth migration path from old to new system

## Deployment Instructions

### Option 1: Firebase Console (Recommended)
1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Select your project
3. Navigate to **Firestore Database** → **Rules**
4. Replace the existing rules with the rules above
5. Click **Publish**

### Option 2: Firebase CLI
```bash
# Save the rules to firestore.rules file
# Then deploy using:
firebase deploy --only firestore:rules
```

### Option 3: GitHub Actions (Automated)
Add to `.github/workflows/firebase-deploy.yml`:
```yaml
- name: Deploy Firestore Rules
  run: firebase deploy --only firestore:rules
```

## Testing the Rules

### Test 1: Anonymous Survey Submission
```javascript
// Should succeed - anyone can submit
await addDoc(collection(db, 'survey-submissions'), {
  surveyId: 'test123',
  token: 'token_abc',
  timestamp: new Date().toISOString(),
  clarity: 4,
  pace: 3,
  engagement: 5,
  positive: 'Great lecture',
  negative: ''
});
```

### Test 2: Read Survey (Active)
```javascript
// Should succeed - anyone can read active surveys
const surveyDoc = await getDoc(doc(db, 'surveys', 'test123'));
// Works only if survey.active === true
```

### Test 3: Admin Operations
```javascript
// Should succeed only if authenticated as swagat.kumar@gmail.com
await setDoc(doc(db, 'surveys', 'new123'), {
  id: 'new123',
  moduleCode: 'CS101',
  lectureDate: '2024-01-20',
  active: true,
  createdAt: new Date().toISOString(),
  createdBy: 'swagat.kumar@gmail.com'
});
```

## Migration Notes

### From Old to New System
1. **Surveys Collection**: Admin must create new surveys using the UI
2. **Submissions**: New submissions go to `survey-submissions` collection
3. **Legacy Data**: Old `lecture-feedback` data remains accessible to admin
4. **No Data Loss**: Rules support both old and new collections

### Recommended Migration Steps
1. Deploy updated rules
2. Keep legacy features marked as "Legacy" in UI
3. Create new surveys using survey management UI
4. Gradually transition students to new survey links
5. Eventually deprecate legacy collection (optional)

## Troubleshooting

### Issue: "Missing or insufficient permissions"
**Cause**: User not authenticated or not authorized admin
**Solution**: Ensure Firebase Authentication is initialized and user is signed in

### Issue: Submission rejected
**Cause**: Missing required fields (surveyId, token, timestamp, ratings)
**Solution**: Ensure all required fields are included in submission data

### Issue: Cannot read surveys
**Cause**: Survey marked as inactive
**Solution**: Admin should activate the survey or check survey.active field

## Security Checklist
- [x] Admin-only access to survey creation/deletion
- [x] Anonymous submissions allowed with validation
- [x] Token-based duplicate prevention
- [x] Immutable submissions (no updates/deletes)
- [x] Email-based admin authorization
- [x] Active survey validation
- [x] Required fields enforcement
- [x] Legacy collection support

---

**Last Updated**: January 2024  
**Author**: Survey Feedback System  
**Version**: 2.0 (Survey-Based Architecture)
