# Session Feedback Application

A modern, real-time feedback collection system designed for educational sessions, lectures, and workshops. Built with Firebase and vanilla JavaScript, featuring anonymous student submissions and comprehensive admin controls.

---

## 📋 Table of Contents

- [Features](#features)
- [Demo Screenshots](#demo-screenshots)
- [Technology Stack](#technology-stack)
- [Setup Instructions](#setup-instructions)
- [Firebase Configuration](#firebase-configuration)
- [Security Rules](#security-rules)
- [Usage Guide](#usage-guide)
- [Troubleshooting](#troubleshooting)
- [License](#license)

---

## ✨ Features

### Student Features
- **Anonymous Feedback Submission**: Students can submit feedback without creating accounts
- **Customizable Forms**: Dynamic forms with admin-defined rating and comment fields
- **Triple-Layer Duplicate Prevention**:
  - Browser session token (localStorage)
  - Device ID (IndexedDB - persists across browser restarts)
  - Browser fingerprint (prevents cross-browser duplicates on same device)
- **User-Friendly Interface**: Clean, responsive design with real-time validation
- **Success Confirmation**: Clear feedback confirmation after submission

### Admin Features
- **Secure Authentication**: Email/password login with Firebase Authentication
- **Survey Management**:
  - Create multiple surveys with unique links
  - Activate/deactivate surveys
  - Customizable survey fields (1-5 rating fields, 1-3 comment fields)
  - Delete individual surveys
- **Real-Time Dashboard**:
  - Three-column responsive layout
  - Live feedback updates (no refresh needed)
  - Overall rating calculation
  - Response count tracking
- **Feedback Management**:
  - View all submissions in organized table
  - Delete individual feedback entries
  - Real-time data synchronization
- **PDF Export**:
  - A4 Landscape format
  - Includes survey title, overall rating, and response count
  - Download timestamp footer
  - Professional formatting (excludes action buttons)
- **Account Management**:
  - Change password
  - Password recovery via email
  - Secure session management

---

## 🖼️ Demo Screenshots

*Add screenshots of your application here*

---

## 🛠️ Technology Stack

- **Frontend**: HTML5, CSS3 (Tailwind CSS), JavaScript (ES6+)
- **Backend**: Firebase (BaaS)
  - Firebase Authentication (Anonymous + Email/Password)
  - Cloud Firestore (NoSQL Database)
- **Libraries**:
  - Firebase SDK v11.6.1
  - jsPDF (PDF generation)
  - html2canvas (Screenshot capture)
  - Tailwind CSS (via CDN)

---

## 🚀 Setup Instructions

### Prerequisites

1. A Google/Firebase account
2. A web browser (Chrome, Firefox, Safari, or Edge)
3. A text editor (VS Code, Sublime, etc.)
4. Basic knowledge of HTML/JavaScript

### Step 1: Clone or Download the Repository

```bash
git clone https://github.com/swagatk/session_feedback.git
cd session_feedback
```

Or download the ZIP file and extract it.

### Step 2: Create a Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click **"Add project"** or **"Create a project"**
3. Enter a project name (e.g., "Session Feedback")
4. (Optional) Enable Google Analytics
5. Click **"Create project"** and wait for setup to complete
6. Click **"Continue"** when ready

### Step 3: Register Your Web App

1. In your Firebase project dashboard, click the **Web icon** (`</>`)
2. Enter an app nickname (e.g., "Feedback Web App")
3. (Optional) Check "Also set up Firebase Hosting" if you plan to use it
4. Click **"Register app"**
5. Copy the Firebase configuration object (you'll need this in Step 6)

### Step 4: Enable Authentication

1. In Firebase Console, go to **"Build" → "Authentication"**
2. Click **"Get started"**
3. Go to the **"Sign-in method"** tab
4. Enable **"Email/Password"**:
   - Click on "Email/Password"
   - Toggle "Enable"
   - Click "Save"
5. Enable **"Anonymous"**:
   - Click on "Anonymous"
   - Toggle "Enable"
   - Click "Save"

### Step 5: Create Admin Account

1. In Firebase Console, go to **"Authentication" → "Users"** tab
2. Click **"Add user"**
3. Enter your admin email (e.g., `admin@example.com`)
4. Enter a strong password
5. Click **"Add user"**

**Important**: Use this email/password to log into the admin panel.

### Step 6: Set Up Cloud Firestore

1. In Firebase Console, go to **"Build" → "Firestore Database"**
2. Click **"Create database"**
3. Choose **"Start in production mode"** (we'll add rules next)
4. Select a Cloud Firestore location (choose closest to your users)
5. Click **"Enable"**

### Step 7: Configure Firestore Security Rules

1. In Firestore Database, go to the **"Rules"** tab
2. Replace the default rules with the following:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Allow anyone to read settings (for student form)
    match /settings/formSettings {
      allow read: if true;
      allow write: if request.auth != null && request.auth.token.email != null;
    }
    
    // Surveys collection
    match /surveys/{surveyId} {
      // Anyone can read active surveys
      allow read: if true;
      // Only authenticated admins can create/update/delete
      allow create, update, delete: if request.auth != null && request.auth.token.email != null;
    }
    
    // Survey submissions collection
    match /survey-submissions/{submissionId} {
      // Anyone can create submissions (anonymous students)
      allow create: if true;
      
      // Only authenticated users can read (for duplicate checking)
      allow read: if request.auth != null;
      
      // Only authenticated admins can delete
      allow delete: if request.auth != null && request.auth.token.email != null;
    }
    
    // Admin settings
    match /settings/adminSettings {
      allow read, write: if request.auth != null && request.auth.token.email != null;
    }
  }
}
```

3. Click **"Publish"**

### Step 8: Create Firestore Composite Indexes

For cross-browser duplicate prevention to work, you need to create two composite indexes:

#### Method 1: Create via Firebase Console

1. Go to **"Firestore Database" → "Indexes"** tab
2. Click **"Create Index"**

**Index 1 (Device ID):**
- Collection ID: `survey-submissions`
- Fields to index:
  - Field: `surveyId`, Order: **Ascending**
  - Field: `deviceId`, Order: **Ascending**
- Query scope: **Collection**
- Click **"Create"**

**Index 2 (Fingerprint):**
- Click **"Create Index"** again
- Collection ID: `survey-submissions`
- Fields to index:
  - Field: `surveyId`, Order: **Ascending**
  - Field: `fingerprint`, Order: **Ascending**
- Query scope: **Collection**
- Click **"Create"**

#### Method 2: Auto-create via Error Link

Alternatively, when you first test the application and try to submit from a different browser, you'll see an error in the browser console with a link to automatically create the required index. Click that link and it will pre-fill the index creation form.

### Step 9: Configure the Application

1. Open `index.html` in your text editor
2. Find the Firebase configuration section (around line 1093-1105):

```javascript
const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_PROJECT_ID.appspot.com",
    messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
    appId: "YOUR_APP_ID"
};
```

3. Replace the placeholder values with your Firebase configuration from Step 3
4. Save the file

### Step 10: Test the Application

1. Open `index.html` in a web browser
2. You should see the student feedback form
3. Test admin login:
   - Click **"Admin Login"** button
   - Enter your admin credentials from Step 5
   - You should see the admin dashboard

---

## 🔧 Firebase Configuration

### Environment Variables (Alternative Setup)

If you're using a build system or want to keep credentials separate, you can use environment variables:

```bash
FIREBASE_API_KEY=your_api_key
FIREBASE_AUTH_DOMAIN=your_auth_domain
FIREBASE_PROJECT_ID=your_project_id
FIREBASE_STORAGE_BUCKET=your_storage_bucket
FIREBASE_MESSAGING_SENDER_ID=your_sender_id
FIREBASE_APP_ID=your_app_id
```

### Firebase Configuration Details

- **apiKey**: Public API key for Firebase (safe to expose in client-side code)
- **authDomain**: Domain for Firebase Authentication
- **projectId**: Your Firebase project identifier
- **storageBucket**: Cloud Storage bucket (optional for this app)
- **messagingSenderId**: For Firebase Cloud Messaging (optional)
- **appId**: Unique identifier for your Firebase app

---

## 🔒 Security Rules

The application uses comprehensive Firestore security rules to protect data:

### Key Security Features

1. **Anonymous users** can:
   - Read active surveys
   - Submit feedback
   - Check for duplicate submissions (read-only)

2. **Authenticated admins** can:
   - Create, update, and delete surveys
   - Read all submissions
   - Delete individual submissions
   - Manage admin settings

3. **Protection mechanisms**:
   - Server-side validation via Firestore rules
   - Client-side duplicate prevention (3 layers)
   - Secure password authentication
   - Password recovery via email

### Important Notes

- Anonymous users cannot modify or delete existing data
- Only email/password authenticated users have admin privileges
- The rules file is located in `FIRESTORE_RULES.md` for reference

---

## 📖 Usage Guide

### For Administrators

#### Creating a Survey

1. Log in to the admin panel
2. In the "Survey Management" section:
   - Enter a module code (e.g., "CS101")
   - Select a lecture date
   - Customize rating fields (1-5 fields)
   - Customize comment fields (1-3 fields)
3. Click **"Create New Survey"**
4. Copy the generated survey link

#### Managing Surveys

- **Activate/Deactivate**: Use the toggle switch next to each survey
- **Copy Link**: Click the 📋 icon to copy the survey link
- **Delete Survey**: Click the 🗑️ icon (with confirmation)

#### Viewing Feedback

1. Select a survey from the dropdown
2. View real-time feedback in the table
3. See overall rating and response count
4. Delete individual entries using the Delete button

#### Exporting Reports

1. Select a survey
2. Click **"Download PDF"**
3. PDF includes:
   - Survey title
   - Overall rating
   - Number of responses
   - All feedback data
   - Download timestamp

#### Account Management

- **Change Password**: Use the "Change Password" section
- **Forgot Password**: Use "Forgot Password?" link on login

### For Students

1. Open the survey link provided by your instructor
2. Fill out the feedback form:
   - Rate each aspect (1-5 scale)
   - Provide comments (optional)
3. Click **"Submit Feedback"**
4. Confirm submission
5. You'll see a success message

**Note**: You can only submit once per survey (prevents duplicates across browsers on the same device).

---

## 🐛 Troubleshooting

### Common Issues

#### "Invalid Survey" Error

**Problem**: Survey link doesn't work  
**Solution**: 
- Ensure the survey is activated (toggle switch is ON)
- Check that the survey hasn't been deleted
- Verify the link was copied correctly

#### "Already Submitted" Message

**Problem**: Student sees this message but hasn't submitted  
**Solution**:
- Check if they submitted from a different browser
- Clear browser data (localStorage and IndexedDB)
- Admin can delete the duplicate entry if it's an error

#### "Database Configuration Error"

**Problem**: Missing Firestore indexes  
**Solution**:
- Follow Step 8 to create composite indexes
- Or click the auto-generated link in browser console

#### PDF Export Not Working

**Problem**: PDF download fails or shows errors  
**Solution**:
- Ensure jsPDF and html2canvas libraries are loaded
- Check browser console for specific errors
- Try a different browser (Chrome/Firefox recommended)

#### Admin Login Failed

**Problem**: Cannot log in with correct credentials  
**Solution**:
- Verify Email/Password authentication is enabled in Firebase
- Check that the admin user exists in Authentication → Users
- Try password reset using "Forgot Password?"
- Check browser console for specific error codes

#### Cross-Browser Duplicate Prevention Not Working

**Problem**: Can submit from different browsers  
**Solution**:
- Ensure Firestore composite indexes are created (Step 8)
- Check browser console for index-related errors
- Wait 1-2 minutes for indexes to build after creation

### Browser Compatibility

**Recommended Browsers**:
- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

**Known Issues**:
- PDF export works best in Chrome/Firefox
- Some older browsers may not support IndexedDB (fallback to fingerprint only)

### Getting Help

If you encounter issues not listed here:

1. Check browser console for error messages (F12)
2. Review Firebase Console for authentication/database errors
3. Verify all setup steps were completed
4. Check `FIRESTORE_RULES.md` for additional configuration details

---

## 📁 Project Structure

```
session_feedback/
├── index.html              # Main application file (SPA)
├── README.md              # This file
└── FIRESTORE_RULES.md     # Firestore security rules reference
```

### Key Sections in index.html

- **Styles**: Tailwind CSS + custom print styles
- **HTML Structure**:
  - Student feedback form
  - Admin login modal
  - Admin dashboard (3-column layout)
  - Custom confirmation dialogs
- **JavaScript**:
  - Firebase initialization
  - Authentication handlers
  - Survey CRUD operations
  - Real-time listeners
  - PDF export logic
  - Duplicate prevention system

---

## 🔄 Future Enhancements

Potential features for future versions:

- [ ] Email notifications when feedback is submitted
- [ ] Export to CSV/Excel
- [ ] Analytics dashboard with charts
- [ ] Multi-language support
- [ ] Mobile app version
- [ ] Bulk survey creation
- [ ] Custom themes/branding
- [ ] Survey templates
- [ ] Response filtering/search

---

## 📝 License

This project is licensed under the MIT License - feel free to use, modify, and distribute as needed.

---

## 👥 Contributing

Contributions are welcome! If you'd like to improve this project:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

---

## 📧 Support

For questions or support:
- Check the Troubleshooting section above
- Review Firebase documentation: https://firebase.google.com/docs
- Check GitHub Issues for similar problems

---

## 🙏 Acknowledgments

- Firebase for providing an excellent backend platform
- Tailwind CSS for the styling framework
- jsPDF and html2canvas for PDF export functionality

---

**Last Updated**: November 3, 2025  
**Version**: 2.0.0