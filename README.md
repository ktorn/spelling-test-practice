# Spelling Test Practice App

A p5js-based web application with Google authentication to help kids practice spelling tests with audio recordings. Features admin/student roles with secure invite system.

## Features

### Admin Features (Parent/Teacher)
- **Google Authentication**: Secure login with Google account
- **Create Batch Mode**: Record yourself saying each spelling word
- **Student Management**: Add student email addresses and generate invite links
- **Batch Management**: Create, organize, and manage spelling batches

### Student Features
- **Secure Access**: Login with Google using invite links from admin
- **Practice Mode**: Practice with randomized word order
- **Audio Playback**: Words play without showing the text
- **Auto-Repeat**: Words automatically repeat after 3 seconds
- **Progress Tracking**: Shows current word and total count

### Security Features
- **Role-Based Access**: Admins can create batches, students can only practice
- **Invite System**: Secure invite links for student access
- **Data Isolation**: Each admin's batches are private to their students
- **Google Authentication**: No passwords to manage

## Setup Instructions

### 1. Firebase Setup (Free Database)

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click "Create a project" or "Add project"
3. Enter project name (e.g., "spelling-test-app")
4. Enable Google Analytics (optional)
5. Click "Create project"

### 2. Enable Required Services

1. **Authentication Setup:**
   - Go to "Authentication" → "Sign-in method"
   - Enable "Google" authentication
   - Add your domain to authorized domains

2. **Database Setup:**
   - Go to "Firestore Database" → "Create database"
   - Choose "Start in test mode" (for development)
   - Select a location close to you

3. **Storage Setup:**
   - Go to "Storage" → "Get started"
   - Choose "Start in test mode"
   - Select the same location as Firestore

### 3. Get Firebase Configuration

1. Go to Project Settings (gear icon) → "General" tab
2. Scroll down to "Your apps" section
3. Click "Web" icon (`</>`) to add a web app
4. Enter app nickname (e.g., "spelling-test")
5. Copy the Firebase configuration object

### 4. Configure Firebase

1. Copy `firebase-config.js.example` to `firebase-config.js`
2. Replace the placeholder values with your actual Firebase config:

```javascript
const firebaseConfig = {
    apiKey: "your-actual-api-key",
    authDomain: "your-project.firebaseapp.com",
    projectId: "your-actual-project-id",
    storageBucket: "your-project.appspot.com",
    messagingSenderId: "your-actual-sender-id",
    appId: "your-actual-app-id"
};
```

**Note:** The `firebase-config.js` file is gitignored to keep your keys private.

### 5. Security Rules

In Firebase Console, go to "Firestore Database" → "Rules" and replace with:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Allow admins to read/write their own data
    match /admins/{adminId} {
      allow read, write: if request.auth != null && request.auth.uid == adminId;
    }
    
    // Allow students to read their own data
    match /students/{studentId} {
      allow read, write: if request.auth != null && request.auth.uid == studentId;
    }
    
    // Allow admins to read/write their batches
    match /batches/{batchId} {
      allow read, write: if request.auth != null && 
        (resource.data.adminUid == request.auth.uid || 
         request.auth.uid in resource.data.studentUids);
    }
    
    // Allow admins to manage invites
    match /invites/{inviteId} {
      allow read, write: if request.auth != null && 
        resource.data.adminUid == request.auth.uid;
    }
  }
}
```

For Storage, go to "Storage" → "Rules" and replace with:

```javascript
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /batches/{batchId}/{fileName} {
      allow read, write: if request.auth != null;
    }
  }
}
```

## Usage

### First Time Setup (Admin)

1. **Open the app** in your web browser
2. **Sign in with Google** - the first user automatically becomes admin
3. **Create your first batch:**
   - Click "Create Batch" mode
   - Enter a word in the text field
   - Click "Record Word" and speak clearly
   - Click "Stop Recording" when done
   - Click "Play" to verify the recording
   - Click "Add to Batch" if satisfied
   - Repeat for all words in the spelling list
   - Click "Save Batch" and enter a name (e.g., "Week 1")

4. **Invite your child:**
   - Enter your child's email address in the "Invite Students" section
   - Click "Add Student"
   - Copy the generated invite link
   - Send the link to your child

### Student Usage

1. **Click the invite link** sent by parent/teacher
2. **Sign in with Google** using the child's Google account
3. **Practice spelling:**
   - Select a batch from the list
   - Click "Next Word" to hear the first word
   - The word will repeat automatically after 3 seconds
   - Continue clicking "Next Word" until all words are practiced
   - Words are randomized each practice session

### Admin Management

- **Create multiple batches** for different weeks or difficulty levels
- **Invite multiple students** by generating separate invite links
- **Manage batches** - all your batches are private to your students
- **Monitor progress** - see which batches are available to students

## Technical Notes

- Audio files are stored in Firebase Storage
- Word lists are stored in Firestore Database
- The app works entirely in the browser (no server needed)
- Requires microphone access for recording
- Works best in modern browsers (Chrome, Firefox, Safari, Edge)

## Troubleshooting

### Microphone Issues
- Ensure browser has microphone permissions
- Try refreshing the page and allowing access again
- Check that no other applications are using the microphone

### Firebase Issues
- Verify your Firebase configuration is correct
- Check that Firestore and Storage rules allow public access
- Ensure your Firebase project is active

### Audio Playback Issues
- Check that audio files were uploaded successfully to Firebase Storage
- Try refreshing the page and selecting the batch again
- Ensure your browser supports audio playback

## Customization

You can easily customize the app by modifying:
- Colors and styling in the `<style>` section
- Button text and labels
- Timing for word repetition (currently 3 seconds)
- Maximum number of words per batch
