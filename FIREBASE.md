# Firebase Storage Setup for CDN File Uploader

This guide explains how to configure Firebase as a storage solution for the **CDN File Uploader**. By following these steps, you'll be able to store your uploaded files on Firebase Storage.

## Prerequisites

1. **Firebase Account**: You need a Firebase account. If you don't have one, you can sign up at [https://firebase.google.com/](https://firebase.google.com/).
2. **Firebase Project**: A Firebase project should be set up in the Firebase Console.

## Steps to Set Up Firebase Storage

### 1. Create a Firebase Project

1. Go to the [Firebase Console](https://console.firebase.google.com/).
2. Click on **Add Project** and follow the steps to create a new Firebase project.
3. Once the project is created, you will be directed to the Firebase project dashboard.

### 2. Set Up Firebase Storage

1. In the Firebase Console, go to the **Storage** section.
2. Click on **Get Started** to enable Firebase Storage for your project.
3. Set your storage rules according to your requirements. For example, to allow unauthenticated uploads, you can set the rules to:

   ```json
   service firebase.storage {
     match /b/{bucket}/o {
       match /{allPaths=**} {
         allow read, write: if true;
       }
     }
   }
   ```

   **Note**: For production, it is recommended to secure your storage rules properly. You can update them to restrict access based on authentication or other criteria.

### 3. Generate a Firebase Admin SDK Key

1. In the Firebase Console, go to **Project Settings**.
2. Select the **Service Accounts** tab.
3. Click **Generate New Private Key**. This will download a JSON file containing your Firebase credentials.
4. Rename this file to `firebase.json` and place it in the root of your project.

### 4. Update `.env` with Firebase Environment Variables

Update your `.env` file with the necessary Firebase environment variables. Make sure you add the following:

```bash
STORAGE=firebase
FIREBASE_STORAGE_BUCKET=gs://your-firebase-bucket
PUBLIC_URL_FIREBASE=https://firebasestorage.googleapis.com/v0/b/your-firebase-bucket/o
```

- Replace `gs://your-firebase-bucket` with the **bucket name** from the Firebase Storage console. It should look something like `gs://your-project-id.appspot.com`.
- `PUBLIC_URL_FIREBASE` is the base URL for accessing your files via Firebase Storage. This URL will be used to build the public URL for accessing uploaded files.

### 5. Install Firebase Admin SDK

You need to install the Firebase Admin SDK to enable Firebase functionality in your Node.js app.

Run the following command:

```bash
npm install firebase-admin
```

### 6. Initialize Firebase in `config/firebase.js`

To initialize Firebase in your project, create a file named `config/firebase.js` where you will set up Firebase using the credentials and bucket you configured.

1. Create a new file in `config/` named `firebase.js`:

```javascript
// config/firebase.js
const admin = require("firebase-admin");
const fireCreds = require("../firebase.json");

let bucket = null;

// Cek apakah aplikasi Firebase sudah diinisialisasi
if (!admin.apps.length) {
  admin.initializeApp({
    credential: admin.credential.cert(fireCreds),
    storageBucket: process.env.FIREBASE_STORAGE_BUCKET,
  });
}

bucket = admin.storage().bucket();

module.exports = bucket;
```

2. This file will initialize Firebase if the `STORAGE=firebase` variable is set in the `.env` file and export the `bucket` object for further use in file upload logic.

3. Import the Firebase bucket in your route or controller where you handle file uploads:

```javascript
// Example: controllers/fileController.js

const bucket = require("../config/firebase.js");

if (process.env.STORAGE === 'firebase') {
  // Your logic for uploading files to Firebase
}
```

### 7. Upload Files to Firebase Storage

Once Firebase is set up, the application will automatically store files in Firebase Storage if `STORAGE=firebase` is set in the `.env` file. When a file is uploaded, it will be uploaded to Firebase Storage and will be accessible via a URL generated by Firebase.

For example, you can access the uploaded file at:

```
https://firebasestorage.googleapis.com/v0/b/your-firebase-bucket/o/your-file-name?alt=media
```

This URL can also be built dynamically in the app using the `PUBLIC_URL_FIREBASE` variable.

### 8. Deploy to Vercel (Optional)

If you want to deploy your project to Vercel, make sure to add the necessary environment variables in the **Vercel dashboard** under the **Environment Variables** section.

The required environment variables are:

- `STORAGE=firebase`
- `FIREBASE_STORAGE_BUCKET`
- `PUBLIC_URL_FIREBASE`
- You can also upload the `firebase.json` key file to Vercel securely.

## Summary

After following the steps outlined above, your application will be configured to store uploaded files in Firebase Storage. Make sure to test the functionality locally before deploying the project to production.

For further assistance, you can refer to the official [Firebase Storage documentation](https://firebase.google.com/docs/storage).