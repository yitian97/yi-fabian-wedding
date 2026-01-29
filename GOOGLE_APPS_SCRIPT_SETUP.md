# Google Apps Script & Google Sheets Setup for RSVP

This guide will help you set up Google Sheets and Google Apps Script to collect RSVP data from your wedding website.

## Step 1: Create a Google Sheet

1. Go to [Google Sheets](https://sheets.google.com)
2. Click **"Blank"** to create a new spreadsheet
3. Name it something like "Wedding RSVPs" or "Yi & Fabian RSVPs"
4. In the first row (Row 1), add these column headers:
   - **A1**: `Timestamp`
   - **B1**: `Name`
   - **C1**: `Phone`
   - **D1**: `Partners` (0 or 1)
   - **E1**: `Kids` (number of kids)
   - **F1**: `Shuttle Bus` (Yes/No)
   - **G1**: `Food Requirements` (optional)
5. Format Row 1 as **bold** (optional, but recommended)
6. **Save the sheet** - remember the name for later

## Step 2: Create Google Apps Script

1. In your Google Sheet, click **Extensions** → **Apps Script**
   - This opens a new tab with the Apps Script editor
2. Delete any default code in the editor
3. Copy and paste this complete code (matches your current form fields):

```javascript
function doPost(e) {
  try {
    // Log that the function was called (this should always appear)
    Logger.log('=== doPost function called ===');
    Logger.log('e.parameter: ' + JSON.stringify(e.parameter));
    
    // Get the active spreadsheet
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    Logger.log('Sheet accessed successfully');
    
    // Get the form data from parameters
    // Google Apps Script automatically parses form data into e.parameter
    // Also check postData in case data comes in POST body
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
    
    var name = params.name || '';
    var phone = params.phone || '';
    var partners = params.partners ? parseInt(params.partners) : 0;
    var kids = params.kids ? parseInt(params.kids) : 0;
    var shuttleBus = params.shuttle_bus || 'No';
    var foodRequirements = params.food_requirements || '';
    
    // Get current timestamp
    var timestamp = new Date();
    
    // Debug: Log what we're receiving (check View > Executions in Apps Script)
    Logger.log('Parsed values:');
    Logger.log('  name: ' + name);
    Logger.log('  phone: ' + phone);
    Logger.log('  partners: ' + partners);
    Logger.log('  kids: ' + kids);
    Logger.log('  shuttleBus: ' + shuttleBus);
    Logger.log('  foodRequirements: ' + foodRequirements);
    
    // Add the data to the sheet in the correct order: Timestamp, Name, Phone, Partners, Kids, Shuttle Bus, Food Requirements
    sheet.appendRow([timestamp, name, phone, partners, kids, shuttleBus, foodRequirements]);
    
    // Send email notification every time someone RSVPs
    // IMPORTANT: Replace "your-email@gmail.com" with your actual email address
    var recipientEmail = "your-email@gmail.com"; // ⬅️ CHANGE THIS TO YOUR EMAIL
    var subject = "🎉 New RSVP: " + name;
    var body = "You have received a new RSVP!\n\n" +
               "Name: " + name + "\n" +
               "Phone: " + phone + "\n" +
               "Partners: " + partners + "\n" +
               "Kids: " + kids + "\n" +
               "Shuttle Bus: " + shuttleBus + "\n" +
               "Food Requirements: " + (foodRequirements || "None") + "\n\n" +
               "Timestamp: " + timestamp;
    MailApp.sendEmail(recipientEmail, subject, body);
    
    // Return success response with CORS headers
    return ContentService.createTextOutput(JSON.stringify({
      "result": "success",
      "message": "RSVP saved successfully!"
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
2. Find the line with `url: 'https://script.google.com/macros/s/...` (around line 213)
3. Replace the URL with your new Web App URL from Step 3
4. Save the file

## Step 5: Test It!

1. Open your website
2. Fill out the RSVP form
3. Submit it
4. Check your Google Sheet - you should see the data appear!

## Step 6: Enable Email Notifications

The script is already set up to send email notifications! You just need to:

1. Go back to your Apps Script editor
2. Find the line that says: `var recipientEmail = "your-email@gmail.com";`
3. **Replace `"your-email@gmail.com"` with your actual email address** (the one you want to receive notifications)
4. Click **Save** (💾 icon)
5. Click **Deploy** → **Manage deployments**
6. Click the edit icon (✏️) next to your deployment
7. Click **Deploy** again (this updates the deployment with the new code)
8. **Authorize email permissions** (if prompted):
   - When you first run the script with email, Google will ask for permission to send emails
   - Click "Review permissions" → Choose your account → Click "Advanced" → "Go to [Project Name] (unsafe)" → Click "Allow"

**Note**: You can add multiple email addresses by separating them with commas:
```javascript
var recipientEmail = "email1@gmail.com, email2@gmail.com";
```

## Troubleshooting

### "Script function not found" error
- Make sure the function is named exactly `doPost` (case-sensitive)
- Make sure you saved the script

### "Access denied" error
- Make sure "Who has access" is set to **Anyone** in the deployment settings
- Make sure you authorized the script when prompted

### No logs appearing in Executions
- **Make sure you've submitted an RSVP** - logs only appear after the script runs
- **Check if the script is being called**: 
  - Open browser Developer Tools (F12) → Console tab
  - Submit the RSVP form
  - Look for any errors or the success message
- **Verify the script URL is correct** in `js/scripts.js`
- **Check if the deployment is active**: Go to Deploy → Manage deployments
- **Try redeploying**: Edit your deployment and click Deploy again

### Data not appearing in sheet
- Check that your column headers match: Timestamp, Name, Phone, Partners, Kids, Shuttle Bus, Food Requirements
- Make sure the sheet is the active one when you created the script
- Check the Apps Script execution log: **View** → **Executions**
- If you see logs but data is wrong, check the log output to see what values were received

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
