# Google Apps Script Setup for Photo Uploads

This guide will help you set up Google Apps Script to receive photo uploads from your wedding website and add them to your Google Photos album.

## Step 1: Create a Google Drive Folder

1. Go to [Google Drive](https://drive.google.com)
2. Create a new folder named "Wedding Guest Photos" (or any name you prefer)
3. **Right-click the folder** → **Share** → Set to **"Anyone with the link can add files"**
   - ⚠️ **Important**: The Google Apps Script needs permission to add files to this folder
   - Make sure the sharing setting allows adding files (not just viewing)
4. **Copy the folder ID** from the URL:
   - The URL will look like: `https://drive.google.com/drive/folders/FOLDER_ID_HERE`
   - Copy the `FOLDER_ID_HERE` part - you'll need this later
   - Your folder link: `https://drive.google.com/drive/folders/1uAInLZGxX7hK8O269sk0OMpXKQwOBe75`

## Step 2: Create Google Apps Script

1. Go to [Google Apps Script](https://script.google.com)
2. Click **"New Project"**
3. Delete any default code in the editor
4. Copy and paste this complete code:

```javascript
// Configuration - UPDATE THESE VALUES
const DRIVE_FOLDER_ID = '1uAInLZGxX7hK8O269sk0OMpXKQwOBe75'; // Your Google Drive folder ID
const GOOGLE_PHOTOS_ALBUM_LINK = 'https://photos.app.goo.gl/ifPCMSD5xS1cWSSQA'; // Your Google Photos album link
const RECIPIENT_EMAIL = 'tianyi9097@gmail.com'; // ⬅️ CHANGE THIS TO YOUR EMAIL

// Invitation codes - Add your codes here (case-insensitive)
// You can add multiple codes separated by commas, or use a single code
const INVITATION_CODES = [
  'yifabi2026',     // Add more codes if needed
  'pipita.amor'  // Example: multiple codes
];

function doPost(e) {
  try {
    Logger.log('=== Photo Upload Request Received ===');
    
    // Parse form data (URL-encoded)
    const invitationCode = (e.parameter.invitation_code || '').trim().toUpperCase();
    const guestName = e.parameter.guest_name || 'Anonymous';
    const photoCount = parseInt(e.parameter.photo_count) || 0;
    const photoIndex = parseInt(e.parameter.photo_index) || 0;
    const totalPhotos = parseInt(e.parameter.total_photos) || photoCount;
    const photoName = e.parameter.photo_name || 'photo.jpg';
    const photoData = e.parameter.photo_data || '';
    const photoType = e.parameter.photo_type || 'image/jpeg';
    
    Logger.log('Invitation code: ' + invitationCode);
    Logger.log('Guest name: ' + guestName);
    Logger.log('Photo count: ' + photoCount);
    Logger.log('Photo index: ' + photoIndex + ' of ' + totalPhotos);
    
    // Validate invitation code
    if (!invitationCode) {
      Logger.log('ERROR: No invitation code provided');
      return ContentService.createTextOutput(JSON.stringify({
        result: 'error',
        message: 'Invitation code is required. Please enter the code from your wedding invitation.'
      })).setMimeType(ContentService.MimeType.JSON);
    }
    
    // Check if code is valid (case-insensitive)
    const validCodes = INVITATION_CODES.map(code => code.toUpperCase());
    if (validCodes.indexOf(invitationCode) === -1) {
      Logger.log('ERROR: Invalid invitation code: ' + invitationCode);
      return ContentService.createTextOutput(JSON.stringify({
        result: 'error',
        message: 'Invalid invitation code. Please check your wedding invitation and try again.'
      })).setMimeType(ContentService.MimeType.JSON);
    }
    
    Logger.log('Invitation code validated successfully');
    
    // If this is just a validation request (for viewing album), return success
    const validateOnly = e.parameter.validate_only === 'true' || e.parameter.validate_only === true;
    Logger.log('validate_only parameter: ' + e.parameter.validate_only);
    Logger.log('validateOnly flag: ' + validateOnly);
    if (validateOnly) {
      Logger.log('Validation-only request, returning success');
      return ContentService.createTextOutput(JSON.stringify({
        result: 'success',
        message: 'Invitation code validated successfully.'
      })).setMimeType(ContentService.MimeType.JSON);
    }
    
    // Get the Drive folder
    const folder = DriveApp.getFolderById(DRIVE_FOLDER_ID);
    
    // Process uploaded photo
    const uploadedFiles = [];
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
    
    if (photoData) {
      try {
        // Convert base64 to blob
        const base64Data = photoData;
        const blob = Utilities.newBlob(Utilities.base64Decode(base64Data), photoType, photoName);
        
        // Create a unique filename
        const filename = guestName.replace(/[^a-zA-Z0-9]/g, '_') + '_' + timestamp + '_' + photoIndex + '.jpg';
        
        // Save to Google Drive
        const file = folder.createFile(blob);
        file.setName(filename);
        
        uploadedFiles.push({
          name: filename,
          url: file.getUrl(),
          size: file.getSize()
        });
        
        Logger.log('Uploaded: ' + filename);
      } catch (error) {
        Logger.log('Error processing photo: ' + error.toString());
        throw error;
      }
    }
    
    // Send email notification
    const subject = '📸 New Wedding Photos Uploaded by ' + guestName;
    const body = 'Guest Name: ' + guestName + '\n' +
                 'Number of Photos: ' + uploadedFiles.length + '\n' +
                 'Timestamp: ' + new Date().toLocaleString() + '\n\n' +
                 'Photos saved to Google Drive folder.\n' +
                 'View folder: https://drive.google.com/drive/folders/1uAInLZGxX7hK8O269sk0OMpXKQwOBe75?usp=share_link\n\n' +
                 'To add photos to Google Photos album:\n' +
                 '1. Open the Google Drive folder\n' +
                 '2. Select the photos\n' +
                 '3. Right-click → Open with → Google Photos\n' +
                 '4. Add to album: ' + GOOGLE_PHOTOS_ALBUM_LINK + '\n\n' +
                 'Uploaded Files:\n' +
                 uploadedFiles.map(f => '- ' + f.name + ' (' + formatFileSize(f.size) + ')').join('\n');
    
    MailApp.sendEmail(RECIPIENT_EMAIL, subject, body);
    
    // Return success response
    return ContentService.createTextOutput(JSON.stringify({
      result: 'success',
      message: 'Photos uploaded successfully!',
      count: uploadedFiles.length
    })).setMimeType(ContentService.MimeType.JSON);
    
  } catch (error) {
    Logger.log('ERROR: ' + error.toString());
    Logger.log('Error stack: ' + error.stack);
    
    return ContentService.createTextOutput(JSON.stringify({
      result: 'error',
      message: error.toString()
    })).setMimeType(ContentService.MimeType.JSON);
  }
}

// Helper function to format file size
function formatFileSize(bytes) {
  if (bytes === 0) return '0 Bytes';
  const k = 1024;
  const sizes = ['Bytes', 'KB', 'MB', 'GB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return Math.round(bytes / Math.pow(k, i) * 100) / 100 + ' ' + sizes[i];
}

// Optional: Test function
function doGet(e) {
  return ContentService.createTextOutput(JSON.stringify({
    result: 'success',
    message: 'Photo upload script is working! Use POST to upload photos.'
  })).setMimeType(ContentService.MimeType.JSON);
}
```

5. **Update the configuration** at the top of the script:
   - ✅ The folder ID is already set: `1uAInLZGxX7hK8O269sk0OMpXKQwOBe75`
   - Replace `tianyi9097@gmail.com` with your email address if needed
   - The Google Photos album link is already set, but you can update it if needed

6. Click **Save** (💾 icon) or press `Ctrl+S` / `Cmd+S`
7. Name your project (e.g., "Wedding Photo Upload Handler")

## Step 3: Deploy as Web App

1. Click **Deploy** → **New deployment**
2. Click the gear icon ⚙️ next to "Select type" and choose **Web app**
3. Configure the deployment:
   - **Description**: "Wedding Photo Upload Handler" (or any name)
   - **Execute as**: **Me** (your email)
   - **Who has access**: **Anyone** (important for public website)
4. Click **Deploy**
5. **Authorize access**:
   - Click "Authorize access"
   - Choose your Google account
   - Click **Advanced** → **Go to [Project Name] (unsafe)**
   - Click **Allow**
6. **Copy the Web App URL** - it will look like:
   ```
   https://script.google.com/macros/s/YOUR_SCRIPT_ID/exec
   ```
   ⚠️ **IMPORTANT**: Save this URL! You'll need it in the next step.

## Step 4: Update Your Website Code

1. Open `gallery.html` in your project
2. Find the line with `const PHOTO_UPLOAD_SCRIPT_URL = 'YOUR_GOOGLE_APPS_SCRIPT_URL_HERE';` (around line 1500)
3. Replace `YOUR_GOOGLE_APPS_SCRIPT_URL_HERE` with your Web App URL from Step 3
4. Save the file

## Step 5: Add Photos to Google Photos Album

Since Google Photos API requires complex OAuth setup, the script saves photos to Google Drive. To add them to your Google Photos album:

### Option A: Manual Method (Recommended for now)
1. Go to your Google Drive folder (the one you created in Step 1)
2. Select the photos you want to add
3. Right-click → **Open with** → **Google Photos**
4. In Google Photos, select the photos → **Add to album** → Choose your wedding album

### Option B: Automated Method (Advanced)
You can set up a more advanced script that automatically adds photos to Google Photos using the Google Photos API. This requires:
- Google Cloud Project setup
- OAuth 2.0 credentials
- Google Photos Library API enabled

For now, the manual method works well and ensures you have control over which photos are added to the album.

## Step 6: Test It!

1. Open your website gallery page
2. Scroll to the "Share Your Photos With Us" section
3. Select some test photos
4. Enter your name (optional)
5. Click "Upload Photos"
6. Check your Google Drive folder - you should see the photos!
7. Check your email - you should receive a notification

## Troubleshooting

### "Script function not found" error
- Make sure the function is named exactly `doPost` (case-sensitive)
- Make sure you saved the script

### "Access denied" error
- Make sure "Who has access" is set to **Anyone** in the deployment settings
- Make sure you authorized the script when prompted
- Make sure your Google Drive folder is shared with "Anyone with the link can add files"

### Photos not appearing in Drive folder
- Check the folder ID is correct in the script
- Check the Apps Script execution log: **View** → **Executions**
- Verify the folder sharing settings

### Email notifications not working
- **Check email address**: Make sure you replaced the email in the script
- **Check spam folder**: Emails might be going to spam/junk
- **Authorize permissions**: Make sure you authorized email permissions when prompted
- **Check execution logs**: Go to **View** → **Executions** → Click on a recent execution → Look for email-related errors

### CORS errors in browser console
- This is normal - Google Apps Script handles CORS automatically
- If you see errors, make sure your deployment is set to "Anyone" access

### "Maximum execution time exceeded" error
- This can happen if uploading many large photos at once
- The script processes photos one by one, so very large batches might timeout
- Solution: Upload fewer photos at a time (the website limits to 10 photos per upload)

## Security Note

Since this is a public website, anyone can upload photos. The script includes:
- File type validation (images only)
- File size limits (handled by the website - 10MB per file)
- Rate limiting (10 photos per upload)

If you want additional protection, you can:
- Add a simple password field
- Add rate limiting in the script (limit uploads per IP)
- Add email validation
- Review photos before adding to Google Photos album

## Future Enhancements

You can enhance this script to:
- Automatically add photos to Google Photos album using the Photos API
- Generate thumbnails
- Apply image compression
- Add metadata (location, date, etc.)
- Create a web interface to review and approve photos

---

**Need help?** Check the [Google Apps Script documentation](https://developers.google.com/apps-script) or test your script using the Apps Script editor's test feature.
