# Google Apps Script Setup for DJ Playlist

This guide will help you set up Google Sheets and Google Apps Script to collect and display song requests (YouTube, Spotify, or SoundCloud links) from your wedding website.

## Step 1: Create a Google Sheet

1. Go to [Google Sheets](https://sheets.google.com)
2. Click **"Blank"** to create a new spreadsheet
3. Name it something like "Wedding Playlist" or "DJ Playlist Requests"
4. In the first row (Row 1), add these column headers:
   - **A1**: `Timestamp`
   - **B1**: `Name`
   - **C1**: `Song Link` (or `YouTube Link` - both work, but `Song Link` is recommended)
5. Format Row 1 as **bold** (optional, but recommended)
6. **Save the sheet** - remember the name for later

## Step 2: Create Google Apps Script

1. In your Google Sheet, click **Extensions** → **Apps Script**
   - This opens a new tab with the Apps Script editor
2. Delete any default code in the editor
3. Copy and paste this complete code:

```javascript
function doPost(e) {
  try {
    // Log that the function was called
    Logger.log('=== doPost function called (Playlist) ===');
    Logger.log('e.parameter: ' + JSON.stringify(e.parameter));
    
    // Get the active spreadsheet
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    Logger.log('Sheet accessed successfully');
    
    // Get the form data from parameters
    var params = e.parameter || {};
    
    // If postData exists, parse it
    if (e.postData && e.postData.contents) {
      var postContents = e.postData.contents;
      var pairs = postContents.split('&');
      for (var i = 0; i < pairs.length; i++) {
        var pair = pairs[i].split('=');
        if (pair.length === 2) {
          params[decodeURIComponent(pair[0])] = decodeURIComponent(pair[1].replace(/\+/g, ' '));
        }
      }
    }
    
    Logger.log('All parameters: ' + JSON.stringify(params));
    
    // Get the playlist data
    // Support both old field name (youtube_link) and new field name (song_link) for backward compatibility
    var name = params.name || 'Anonymous';
    var song_link = params.song_link || params.youtube_link || '';
    var timestamp = params.timestamp || new Date().toISOString();
    
    // Validate song link
    if (!song_link || song_link.trim() === '') {
      return ContentService.createTextOutput(JSON.stringify({
        "result": "error",
        "message": "Song link is required"
      })).setMimeType(ContentService.MimeType.JSON);
    }
    
    // Validate URL is from supported platforms (YouTube, Spotify, or SoundCloud)
    if (!song_link.match(/(?:youtube\.com|youtu\.be|spotify\.com|soundcloud\.com)/)) {
      return ContentService.createTextOutput(JSON.stringify({
        "result": "error",
        "message": "Please enter a valid YouTube, Spotify, or SoundCloud link"
      })).setMimeType(ContentService.MimeType.JSON);
    }
    
    // Get current timestamp for sheet
    var sheetTimestamp = new Date();
    
    // Debug: Log what we're receiving
    Logger.log('Parsed values:');
    Logger.log('  name: ' + name);
    Logger.log('  song_link: ' + song_link);
    Logger.log('  timestamp: ' + timestamp);
    
    // Add the data to the sheet in the correct order: Timestamp, Name, Song Link
    sheet.appendRow([sheetTimestamp, name, song_link]);
    
    // Send email notification (optional)
    // IMPORTANT: Replace "your-email@gmail.com" with your actual email address
    var recipientEmail = "tianyi9097@gmail.com"; // ⬅️ CHANGE THIS TO YOUR EMAIL
    var subject = "🎵 New Song Request: " + (name !== 'Anonymous' ? name : 'Anonymous Guest');
    var body = "You have received a new song request on your wedding website!\n\n" +
               "Name: " + name + "\n" +
               "Song Link: " + song_link + "\n\n" +
               "Timestamp: " + sheetTimestamp;
    MailApp.sendEmail(recipientEmail, subject, body);
    
    // Return success response with CORS headers
    return ContentService.createTextOutput(JSON.stringify({
      "result": "success",
      "message": "Song request saved successfully!"
    })).setMimeType(ContentService.MimeType.JSON);
    
  } catch (error) {
    // Log the error
    Logger.log('ERROR: ' + error.toString());
    Logger.log('Error stack: ' + error.stack);
    
    // Return error response
    return ContentService.createTextOutput(JSON.stringify({
      "result": "error",
      "message": error.toString()
    })).setMimeType(ContentService.MimeType.JSON);
  }
}

// Function to retrieve songs (for displaying on website)
function doGet(e) {
  try {
    // Get the active spreadsheet
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    
    // Get all data (skip header row)
    var data = sheet.getDataRange().getValues();
    
    // Skip the first row (headers)
    var songs = [];
    for (var i = 1; i < data.length; i++) {
      var row = data[i];
      if (row[0] && row[2]) { // Make sure timestamp and song link exist
        songs.push({
          timestamp: row[0].toString(),
          name: row[1] || 'Anonymous',
          song_link: row[2],
          youtube_link: row[2] // Keep for backward compatibility
        });
      }
    }
    
    // Sort by timestamp (newest first)
    songs.sort(function(a, b) {
      return new Date(b.timestamp) - new Date(a.timestamp);
    });
    
    // Return songs as JSON
    return ContentService.createTextOutput(JSON.stringify({
      "result": "success",
      "songs": songs
    })).setMimeType(ContentService.MimeType.JSON);
    
  } catch (error) {
    Logger.log('ERROR in doGet: ' + error.toString());
    return ContentService.createTextOutput(JSON.stringify({
      "result": "error",
      "message": error.toString()
    })).setMimeType(ContentService.MimeType.JSON);
  }
}
```

4. Click **Save** (💾 icon) or press `Ctrl+S` / `Cmd+S`
5. Name your project (e.g., "Wedding Playlist Handler")

## Step 3: Deploy as Web App

1. Click **Deploy** → **New deployment**
2. Click the gear icon ⚙️ next to "Select type" and choose **Web app**
3. Configure the deployment:
   - **Description**: "Wedding Playlist Handler" (or any name)
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

1. Open `memories.html` in your project
2. Find the JavaScript section with the playlist form handler (around line 775+)
3. Look for the line that says:
   ```javascript
   var scriptUrl = 'https://script.google.com/macros/s/YOUR_PLAYLIST_SCRIPT_ID/exec';
   ```
4. Replace `YOUR_PLAYLIST_SCRIPT_ID` with your actual Web App URL from Step 3

   For example, if your URL is:
   ```
   https://script.google.com/macros/s/AKfycbzd8si22j1o2GqeU1zd3RPKE9w9ZvluGG_vBT8hTCPKb5E8zrk7BPkCG4Cu0URuM4mK/exec
   ```
   
   Then replace the line to:
   ```javascript
   var scriptUrl = 'https://script.google.com/macros/s/AKfycbzd8si22j1o2GqeU1zd3RPKE9w9ZvluGG_vBT8hTCPKb5E8zrk7BPkCG4Cu0URuM4mK/exec';
   ```

5. Do this in **two places**:
   - In the `loadPlaylist()` function (around line 650+)
   - In the form submission handler (around line 775+)

## Step 5: Enable Email Notifications

1. Go back to your Apps Script editor
2. Find the line that says: `var recipientEmail = "tianyi9097@gmail.com";`
3. **Replace with your actual email address** (if different)
4. Click **Save** (💾 icon)
5. Click **Deploy** → **Manage deployments**
6. Click the edit icon (✏️) next to your deployment
7. Click **Deploy** again (this updates the deployment with the new code)
8. **Authorize email permissions** (if prompted)

## Step 6: Test It!

1. Open your website
2. Go to the Memories page → DJ Playlist section
3. Fill out the form with a YouTube link and submit
4. Check your Google Sheet - you should see the data appear!
5. Check your email - you should receive a notification!
6. The YouTube video should appear embedded in the playlist display!

## Troubleshooting

### "Script function not found" error
- Make sure the functions are named exactly `doPost` and `doGet` (case-sensitive)
- Make sure you saved the script

### "Access denied" error
- Make sure "Who has access" is set to **Anyone** in the deployment settings
- Make sure you authorized the script when prompted

### Songs not appearing on website
- Check browser console (F12) for errors
- Verify the script URL is correct in `memories.html` (check both places!)
- Make sure the deployment is active: Go to Deploy → Manage deployments
- Check the Apps Script execution log: **View** → **Executions**

### Data not appearing in sheet
- Check that your column headers match: Timestamp, Name, Song Link (or YouTube Link)
- Make sure the sheet is the active one when you created the script
- Check the Apps Script execution log: **View** → **Executions**

### Songs not embedding properly
- **YouTube**: Make sure the link is in a valid format (youtube.com or youtu.be)
- **Spotify**: Supports track, album, and playlist links (e.g., spotify.com/track/...)
- **SoundCloud**: Make sure the link is a valid SoundCloud URL
- Check that the link is being saved correctly in the Google Sheet
- The embed extraction happens automatically in the JavaScript for all supported platforms

---

**Note**: The script handles both saving song requests (POST) and retrieving them (GET), so your website can display all song requests from the Google Sheet with embedded players for YouTube, Spotify, and SoundCloud!
