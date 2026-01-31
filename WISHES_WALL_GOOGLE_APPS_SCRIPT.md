# Google Apps Script Setup for Wishes Wall

This guide will help you set up Google Sheets and Google Apps Script to collect and display wishes from your wedding website.

## Step 1: Create a Google Sheet

1. Go to [Google Sheets](https://sheets.google.com)
2. Click **"Blank"** to create a new spreadsheet
3. Name it something like "Wedding Wishes" or "Yi & Fabian Wishes"
4. In the first row (Row 1), add these column headers:
   - **A1**: `Timestamp`
   - **B1**: `Name`
   - **C1**: `Message`
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
    Logger.log('=== doPost function called (Wishes Wall) ===');
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
    
    // Get the wish data
    var name = params.name || 'Anonymous';
    var message = params.message || '';
    var timestamp = params.timestamp || new Date().toISOString();
    
    // Validate message
    if (!message || message.trim() === '') {
      return ContentService.createTextOutput(JSON.stringify({
        "result": "error",
        "message": "Message is required"
      })).setMimeType(ContentService.MimeType.JSON);
    }
    
    // Get current timestamp for sheet
    var sheetTimestamp = new Date();
    
    // Debug: Log what we're receiving
    Logger.log('Parsed values:');
    Logger.log('  name: ' + name);
    Logger.log('  message: ' + message);
    Logger.log('  timestamp: ' + timestamp);
    
    // Add the data to the sheet in the correct order: Timestamp, Name, Message
    sheet.appendRow([sheetTimestamp, name, message]);
    
    // Send email notification (optional)
    // IMPORTANT: Replace "your-email@gmail.com" with your actual email address
    var recipientEmail = "tianyi9097@gmail.com"; // ⬅️ CHANGE THIS TO YOUR EMAIL
    var subject = "💕 New Wish: " + (name !== 'Anonymous' ? name : 'Anonymous Guest');
    var body = "You have received a new wish on your wedding website!\n\n" +
               "Name: " + name + "\n" +
               "Message: " + message + "\n\n" +
               "Timestamp: " + sheetTimestamp;
    MailApp.sendEmail(recipientEmail, subject, body);
    
    // Return success response with CORS headers
    return ContentService.createTextOutput(JSON.stringify({
      "result": "success",
      "message": "Wish saved successfully!"
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

// Function to retrieve wishes (for displaying on website)
function doGet(e) {
  try {
    // Get the active spreadsheet
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    
    // Get all data (skip header row)
    var data = sheet.getDataRange().getValues();
    
    // Skip the first row (headers)
    var wishes = [];
    for (var i = 1; i < data.length; i++) {
      var row = data[i];
      if (row[0] && row[2]) { // Make sure timestamp and message exist
        wishes.push({
          timestamp: row[0].toString(),
          name: row[1] || 'Anonymous',
          message: row[2]
        });
      }
    }
    
    // Sort by timestamp (newest first)
    wishes.sort(function(a, b) {
      return new Date(b.timestamp) - new Date(a.timestamp);
    });
    
    // Return wishes as JSON
    return ContentService.createTextOutput(JSON.stringify({
      "result": "success",
      "wishes": wishes
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
5. Name your project (e.g., "Wedding Wishes Handler")

## Step 3: Deploy as Web App

1. Click **Deploy** → **New deployment**
2. Click the gear icon ⚙️ next to "Select type" and choose **Web app**
3. Configure the deployment:
   - **Description**: "Wedding Wishes Handler" (or any name)
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
2. Find the JavaScript section with the wishes form handler (around line 600+)
3. Look for the commented code that says `// TODO: Submit to Google Apps Script`
4. Replace `YOUR_GOOGLE_APPS_SCRIPT_URL` with your Web App URL from Step 3
5. Uncomment the AJAX code block
6. Also update the `loadWishes()` function to fetch from Google Apps Script

Here's the updated code to replace in `memories.html`:

```javascript
// Load wishes from Google Apps Script
function loadWishes() {
    // Replace YOUR_GOOGLE_APPS_SCRIPT_URL with your actual URL
    var scriptUrl = 'YOUR_GOOGLE_APPS_SCRIPT_URL';
    
    $.ajax({
        url: scriptUrl,
        type: 'GET',
        dataType: 'json',
        success: function(response) {
            if (response.result === 'success' && response.wishes) {
                displayWishes(response.wishes);
            } else {
                // Fallback to localStorage
                var wishes = JSON.parse(localStorage.getItem('weddingWishes') || '[]');
                displayWishes(wishes);
            }
        },
        error: function() {
            // Fallback to localStorage if script fails
            var wishes = JSON.parse(localStorage.getItem('weddingWishes') || '[]');
            displayWishes(wishes);
        }
    });
}

// In the form submission handler, replace the TODO section with:
$.ajax({
    url: 'YOUR_GOOGLE_APPS_SCRIPT_URL/exec', // Replace with your URL
    type: 'POST',
    data: {
        name: wish.name,
        message: wish.message,
        timestamp: wish.timestamp
    },
    dataType: 'json',
    success: function(response) {
        if (response.result === 'success') {
            $('#wishes-alert-wrapper').html(showAlert('success', '<strong>Thank you!</strong> Your wish has been submitted.'));
            $('#wishes-form')[0].reset();
            loadWishes(); // Reload wishes from server
        } else {
            $('#wishes-alert-wrapper').html(showAlert('error', response.message || 'There was an error submitting your wish.'));
        }
    },
    error: function() {
        // Fallback to localStorage if script fails
        $('#wishes-alert-wrapper').html(showAlert('success', '<strong>Thank you!</strong> Your wish has been saved locally.'));
        $('#wishes-form')[0].reset();
        displayWishes(wishes);
    }
});
```

## Step 5: Enable Email Notifications

1. Go back to your Apps Script editor
2. Find the line that says: `var recipientEmail = "your-email@gmail.com";`
3. **Replace `"your-email@gmail.com"` with your actual email address**
4. Click **Save** (💾 icon)
5. Click **Deploy** → **Manage deployments**
6. Click the edit icon (✏️) next to your deployment
7. Click **Deploy** again (this updates the deployment with the new code)
8. **Authorize email permissions** (if prompted)

## Step 6: Test It!

1. Open your website
2. Go to the Memories page → Wishes Wall section
3. Fill out the form and submit a wish
4. Check your Google Sheet - you should see the data appear!
5. Check your email - you should receive a notification!

## Troubleshooting

### "Script function not found" error
- Make sure the functions are named exactly `doPost` and `doGet` (case-sensitive)
- Make sure you saved the script

### "Access denied" error
- Make sure "Who has access" is set to **Anyone** in the deployment settings
- Make sure you authorized the script when prompted

### Wishes not appearing on website
- Check browser console (F12) for errors
- Verify the script URL is correct in `memories.html`
- Make sure the deployment is active: Go to Deploy → Manage deployments
- Check the Apps Script execution log: **View** → **Executions**

### Data not appearing in sheet
- Check that your column headers match: Timestamp, Name, Message
- Make sure the sheet is the active one when you created the script
- Check the Apps Script execution log: **View** → **Executions**

---

**Note**: The script handles both saving wishes (POST) and retrieving them (GET), so your website can display all wishes from the Google Sheet!
