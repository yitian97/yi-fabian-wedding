# Google Apps Script & Google Sheets Setup for RSVP

This guide will help you set up Google Sheets and Google Apps Script to collect RSVP data from your wedding website.

## Step 1: Create a Google Sheet

1. Go to [Google Sheets](https://sheets.google.com)
2. Click **"Blank"** to create a new spreadsheet
3. Name it something like "Wedding RSVPs" or "Yi & Fabian RSVPs"
4. In the first row (Row 1), add these column headers:
   - **A1**: `Timestamp`
   - **B1**: `Email`
   - **C1**: `Name`
   - **D1**: `Extras` (number of additional guests)
5. Format Row 1 as **bold** (optional, but recommended)
6. **Save the sheet** - remember the name for later

## Step 2: Create Google Apps Script

1. In your Google Sheet, click **Extensions** → **Apps Script**
   - This opens a new tab with the Apps Script editor
2. Delete any default code in the editor
3. Copy and paste this code:

```javascript
function doPost(e) {
  try {
    // Get the active spreadsheet
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    
    // Get the form data
    var email = e.parameter.email;
    var name = e.parameter.name;
    var extras = e.parameter.extras;
    
    // Get current timestamp
    var timestamp = new Date();
    
    // Add the data to the sheet
    sheet.appendRow([timestamp, email, name, extras]);
    
    // Optional: Send email notification (uncomment if you want this)
    // var subject = "New RSVP: " + name;
    // var body = "Name: " + name + "\nEmail: " + email + "\nAdditional Guests: " + extras;
    // MailApp.sendEmail("your-email@gmail.com", subject, body);
    
    // Return success response with CORS headers
    return ContentService.createTextOutput(JSON.stringify({
      "result": "success",
      "message": "RSVP saved successfully!"
    })).setMimeType(ContentService.MimeType.JSON);
    
  } catch (error) {
    // Return error response
    return ContentService.createTextOutput(JSON.stringify({
      "result": "error",
      "message": error.toString()
    })).setMimeType(ContentService.MimeType.JSON);
  }
}

// Optional: Add doGet for testing
function doGet(e) {
  return ContentService.createTextOutput(JSON.stringify({
    "result": "success",
    "message": "Script is working! Use POST to submit RSVP."
  })).setMimeType(ContentService.MimeType.JSON);
}
```

4. Click **Save** (💾 icon) or press `Ctrl+S` / `Cmd+S`
5. Name your project (e.g., "Wedding RSVP Handler")

## Step 3: Deploy as Web App

1. Click **Deploy** → **New deployment**
2. Click the gear icon ⚙️ next to "Select type" and choose **Web app**
3. Configure the deployment:
   - **Description**: "Wedding RSVP Handler" (or any name)
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

1. Open `js/scripts.js` in your project
2. Find line 212 (the line with `$.post('https://script.google.com...`)
3. Replace the URL with your new Web App URL from Step 3
4. Save the file

## Step 5: Test It!

1. Open your website
2. Fill out the RSVP form
3. Submit it
4. Check your Google Sheet - you should see the data appear!

## Optional: Email Notifications

If you want to receive an email every time someone RSVPs:

1. Go back to your Apps Script editor
2. Find the commented lines (starting with `// Optional: Send email notification`)
3. Uncomment those lines (remove the `//`)
4. Replace `"your-email@gmail.com"` with your actual email address
5. Click **Save**
6. Click **Deploy** → **Manage deployments**
7. Click the edit icon (✏️) next to your deployment
8. Click **Deploy** again (this updates the deployment with the new code)

## Troubleshooting

### "Script function not found" error
- Make sure the function is named exactly `doPost` (case-sensitive)
- Make sure you saved the script

### "Access denied" error
- Make sure "Who has access" is set to **Anyone** in the deployment settings
- Make sure you authorized the script when prompted

### Data not appearing in sheet
- Check that your column headers match: Timestamp, Email, Name, Extras
- Make sure the sheet is the active one when you created the script
- Check the Apps Script execution log: **View** → **Executions**

### CORS errors in browser console
- This is normal - Google Apps Script handles CORS automatically
- If you see errors, make sure your deployment is set to "Anyone" access

## Security Note

Since you removed invite codes, anyone can submit RSVPs. If you want to add some protection later, you can:
- Add a simple password field
- Add rate limiting in the script
- Add email validation

---

**Need help?** Check the [Google Apps Script documentation](https://developers.google.com/apps-script) or test your script using the Apps Script editor's test feature.
