# Google Sheets Integration Guide

## Issue: Integrate Waiting List Form with Google Sheets

### Problem Statement

Currently, the waiting list form uses Formspree for collecting form submissions. To better manage and analyze the waiting list data, we need to integrate the form directly with Google Sheets, allowing automatic recording of submissions in a spreadsheet.

### Benefits

- **Direct Data Access**: View and manage submissions in real-time within Google Sheets
- **Data Analysis**: Use Google Sheets' built-in tools for sorting, filtering, and analyzing waiting list data
- **Collaboration**: Share the spreadsheet with team members for collaborative management
- **Automation**: Leverage Google Apps Script for automated workflows (email notifications, data validation, etc.)
- **Cost-Effective**: Free for up to reasonable usage limits
- **Export Options**: Easy export to CSV, Excel, or other formats

---

## Step-by-Step Implementation Guide

### Phase 1: Google Sheets Setup

#### Step 1: Create a New Google Sheet

1. Go to [Google Sheets](https://sheets.google.com)
2. Click on **"Blank"** to create a new spreadsheet
3. Name your spreadsheet: `Speech Therapy Waiting List`

#### Step 2: Set Up Column Headers

Create the following columns in Row 1 (these match the form fields):

| Column | Header Name | Description |
|--------|-------------|-------------|
| A | Timestamp | Automatic timestamp of submission |
| B | Patient Name | Name of the patient |
| C | Relationship | Relationship to patient (Parent, Self, Caregiver, etc.) |
| D | Phone Number | Contact phone number |
| E | Email | Contact email address |
| F | Age Group | Patient's age group |
| G | Concerns | Speech/communication concerns |
| H | Preferred Time | Preferred appointment time slot |
| I | One-Time Meeting | Preference for one-time meeting |
| J | Consent Given | Consent checkbox status |
| K | Language | Form language (Hebrew/English) |
| L | Submission ID | Unique submission identifier |

**Recommended:** Apply header formatting:
- Bold text
- Background color (light blue or gray)
- Freeze the first row (View → Freeze → 1 row)

#### Step 3: Set Up Data Validation (Optional but Recommended)

1. Select the **Age Group** column (F2:F1000)
2. Click **Data → Data validation**
3. Set criteria to "List of items"
4. Enter: `0-3,4-6,7-12,13-17,18+`
5. Click "Save"

Repeat for other columns with specific values:
- **Preferred Time**: `morning,afternoon,evening,flexible`
- **One-Time Meeting**: `yes,no`
- **Language**: `he,en`

---

### Phase 2: Google Apps Script Setup

#### Step 4: Create Apps Script Project

1. In your Google Sheet, click **Extensions → Apps Script**
2. Delete any default code in the editor
3. Name your project: `Waiting List Form Handler`

#### Step 5: Add the Web App Script

Copy and paste the following code into the Apps Script editor:

```javascript
/**
 * Speech Therapy Waiting List - Google Sheets Integration
 * This script receives form submissions via POST requests and saves them to the spreadsheet
 */

// Configuration
const SHEET_NAME = 'Sheet1'; // Change if you renamed your sheet
const EXPECTED_SECRET_KEY = 'YOUR_SECRET_KEY_HERE'; // Change this to a random secret

/**
 * Handle POST requests from the waiting list form
 */
function doPost(e) {
  try {
    // Log the incoming request for debugging
    Logger.log('Received POST request');
    Logger.log('Request body: ' + e.postData.contents);
    
    // Parse the JSON payload
    const data = JSON.parse(e.postData.contents);
    
    // Verify secret key for basic security
    if (data.secretKey !== EXPECTED_SECRET_KEY) {
      return ContentService.createTextOutput(JSON.stringify({
        'status': 'error',
        'message': 'Invalid secret key'
      })).setMimeType(ContentService.MimeType.JSON);
    }
    
    // Get the active spreadsheet and sheet
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = ss.getSheetByName(SHEET_NAME);
    
    if (!sheet) {
      throw new Error('Sheet not found: ' + SHEET_NAME);
    }
    
    // Prepare row data
    const timestamp = new Date();
    const rowData = [
      timestamp,                          // Timestamp
      data.patientName || '',             // Patient Name
      data.relationship || '',            // Relationship
      data.phone || '',                   // Phone Number
      data.email || '',                   // Email
      data.ageGroup || '',                // Age Group
      data.concerns || '',                // Concerns
      data.preferredTime || '',           // Preferred Time
      data.oneTimeMeeting || '',          // One-Time Meeting
      data.consentGiven ? 'Yes' : 'No',   // Consent Given
      data.language || 'he',              // Language
      data.submissionId || ''             // Submission ID
    ];
    
    // Append the data to the sheet
    sheet.appendRow(rowData);
    
    // Optional: Send email notification
    sendNotificationEmail(data, timestamp);
    
    // Return success response
    return ContentService.createTextOutput(JSON.stringify({
      'status': 'success',
      'message': 'Form submitted successfully',
      'timestamp': timestamp.toISOString()
    })).setMimeType(ContentService.MimeType.JSON);
    
  } catch (error) {
    // Log error for debugging
    Logger.log('Error: ' + error.toString());
    
    // Return error response
    return ContentService.createTextOutput(JSON.stringify({
      'status': 'error',
      'message': error.toString()
    })).setMimeType(ContentService.MimeType.JSON);
  }
}

/**
 * Handle GET requests (for testing)
 */
function doGet(e) {
  return ContentService.createTextOutput(JSON.stringify({
    'status': 'ok',
    'message': 'Google Sheets integration is active'
  })).setMimeType(ContentService.MimeType.JSON);
}

/**
 * Send email notification when a new submission is received
 */
function sendNotificationEmail(data, timestamp) {
  try {
    // Configure your notification email here
    const recipientEmail = 'YOUR_EMAIL@example.com'; // Change this
    const subject = 'New Waiting List Submission - ' + data.patientName;
    
    const body = `
A new waiting list submission has been received:

Timestamp: ${timestamp.toLocaleString('en-US', { timeZone: 'Asia/Jerusalem' })}
Patient Name: ${data.patientName}
Relationship: ${data.relationship}
Phone: ${data.phone}
Email: ${data.email}
Age Group: ${data.ageGroup}
Concerns: ${data.concerns}
Preferred Time: ${data.preferredTime}
One-Time Meeting: ${data.oneTimeMeeting}
Language: ${data.language === 'he' ? 'Hebrew' : 'English'}

View the spreadsheet: ${SpreadsheetApp.getActiveSpreadsheet().getUrl()}
    `;
    
    // Uncomment the line below to enable email notifications
    // MailApp.sendEmail(recipientEmail, subject, body);
    
  } catch (error) {
    Logger.log('Email notification error: ' + error.toString());
    // Don't throw error - email notification failure shouldn't block the submission
  }
}

/**
 * Test function to verify the script works
 */
function testSubmission() {
  const testData = {
    secretKey: EXPECTED_SECRET_KEY,
    patientName: 'Test Patient',
    relationship: 'parent',
    phone: '050-1234567',
    email: 'test@example.com',
    ageGroup: '4-6',
    concerns: 'Test concerns',
    preferredTime: 'morning',
    oneTimeMeeting: 'yes',
    consentGiven: true,
    language: 'he',
    submissionId: 'test-' + new Date().getTime()
  };
  
  const e = {
    postData: {
      contents: JSON.stringify(testData)
    }
  };
  
  const result = doPost(e);
  Logger.log('Test result: ' + result.getContent());
}
```

#### Step 6: Configure the Script

1. **Set a Secret Key**: 
   - Find the line: `const EXPECTED_SECRET_KEY = 'YOUR_SECRET_KEY_HERE';`
   - Replace with a strong random string (e.g., `'wl_secret_a9f8e7d6c5b4a3f2e1d0c9b8a7f6e5d4'`)
   - Generate a secure key using: `openssl rand -hex 32` or an online UUID generator
   - Keep this key secure - you'll need it for the web form

2. **Configure Email Notifications** (Optional):
   - Find the line: `const recipientEmail = 'YOUR_EMAIL@example.com';`
   - Replace with your actual email address (e.g., `admin@yourcompany.com`)
   - Uncomment the `MailApp.sendEmail()` line to enable notifications

3. **Verify Sheet Name**:
   - If you renamed your sheet from "Sheet1", update the `SHEET_NAME` constant

#### Step 7: Deploy the Web App

1. Click the **Deploy** button (top right) → **New deployment**
2. Click the gear icon ⚙️ next to "Select type" → Select **Web app**
3. Configure the deployment:
   - **Description**: `Initial deployment for waiting list form`
   - **Execute as**: `Me (your@email.com)`
   - **Who has access**: `Anyone` (required for public form access)
4. Click **Deploy**
5. **Authorize the app**:
   - Click "Authorize access"
   - Choose your Google account
   - Click "Advanced" → "Go to Waiting List Form Handler (unsafe)"
   - Click "Allow"
6. **Copy the Web App URL**:
   - You'll see a URL like: `https://script.google.com/macros/s/ABC123.../exec`
   - **Save this URL** - you'll need it for the form integration

#### Step 8: Test the Deployment

1. In the Apps Script editor, select the `testSubmission` function from the dropdown
2. Click the **Run** button (▶️)
3. Check your Google Sheet - you should see a new test row added
4. Check **Execution log** (View → Logs) for any errors

---

### Phase 3: Update the Web Form

#### Step 9: Update the Form Submission Code

In your `index.html` file, find the `handleSubmit` function (around line 2312) and replace the Formspree integration with the Google Sheets integration:

**Replace this:**
```javascript
const response = await fetch('https://formspree.io/f/YOUR_FORM_ID', {
    method: 'POST',
    headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(formData)
});
```

**With this:**
```javascript
// Google Sheets Web App URL (replace with your deployment URL)
const GOOGLE_SHEETS_URL = 'https://script.google.com/macros/s/ABC123.../exec';
const SECRET_KEY = 'wl_secret_a9f8e7d6c5b4a3f2e1d0c9b8a7f6e5d4'; // Must match the key in Apps Script

// Prepare data for Google Sheets
const submissionData = {
    secretKey: SECRET_KEY,
    patientName: formData.patientName,
    relationship: formData.relationship,
    phone: formData.phone,
    email: formData.email,
    ageGroup: formData.ageGroup,
    concerns: formData.concerns,
    preferredTime: formData.preferredTime,
    oneTimeMeeting: formData.oneTimeMeeting,
    consentGiven: formData.consentGiven,
    language: state.currentLanguage,
    submissionId: formData.submittedAt
};

const response = await fetch(GOOGLE_SHEETS_URL, {
    method: 'POST',
    mode: 'no-cors', // Required for Google Apps Script
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(submissionData)
});
```

**Important Security Notes:**
- ⚠️ **Secret Key Exposure**: The secret key will be visible in client-side code. This provides basic protection against casual abuse but is not cryptographically secure. For a public form, this is an acceptable trade-off.
- The secret key primarily prevents automated spam and accidental submissions to your sheet
- For higher security needs, consider implementing reCAPTCHA or server-side validation
- Replace `GOOGLE_SHEETS_URL` with your actual Web App URL from Step 7
- Replace `SECRET_KEY` with the same secret key you set in Apps Script (use a strong random string)
- The `mode: 'no-cors'` is required for Google Apps Script endpoints but prevents reading response details

#### Step 10: Update Success Message Handling

Since `no-cors` mode prevents reading the response, update the success handling:

```javascript
// After the fetch call
try {
    await fetch(GOOGLE_SHEETS_URL, {
        method: 'POST',
        mode: 'no-cors',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(submissionData)
    });
    
    // With no-cors mode, we can't verify the response
    // Show success message if no error was thrown
    showSuccessMessage();
    clearFormData();
    
} catch (error) {
    // Handle network errors
    showErrorMessage();
    console.error('Submission error:', error);
}

function showSuccessMessage() {
    const successMessage = state.currentLanguage === 'he' 
        ? 'תודה! הצטרפת בהצלחה לרשימת ההמתנה' 
        : 'Thank you! You have successfully joined the waiting list';
    
    alert(successMessage);
    // Reset form or redirect to confirmation page
}

function showErrorMessage() {
    const errorMessage = state.currentLanguage === 'he'
        ? 'אירעה שגיאה בשליחת הטופס. אנא נסו שוב.'
        : 'An error occurred submitting the form. Please try again.';
    
    alert(errorMessage);
}
```

**Limitations with no-cors mode:**
- ⚠️ **Cannot validate response**: The browser won't let you read the response body
- ⚠️ **Limited error detection**: You'll only catch network errors, not submission errors
- Success is assumed if the fetch doesn't throw an error
- **Mitigation**: Monitor your Google Sheet for submissions and check Apps Script logs for errors
- **Alternative**: Implement server-side validation or use a backend proxy if response validation is critical
}
```

---

### Phase 4: Testing & Verification

#### Step 11: Test the Complete Integration

1. **Open your waiting list form** in a browser
2. **Fill out all form fields** with test data
3. **Submit the form**
4. **Check your Google Sheet** - a new row should appear within a few seconds
5. **Verify all data** is correctly recorded

**Troubleshooting Tips:**

| Issue | Solution |
|-------|----------|
| No data appears in sheet | Check the Apps Script execution log for errors |
| "Invalid secret key" error | Ensure the secret key matches in both the form and Apps Script |
| CORS errors | Verify you're using `mode: 'no-cors'` in the fetch request |
| Data appears in wrong columns | Check the order of fields in the `rowData` array |
| Email notifications not working | Uncomment the `MailApp.sendEmail()` line and re-test |

#### Step 12: Monitor Submissions

1. **Check the spreadsheet** regularly for new submissions
2. **Review the Apps Script logs**:
   - In Apps Script: View → Executions
   - Check for any failed executions or errors
3. **Set up alerts** (optional):
   - In Google Sheets: Tools → Notification rules
   - Get notified when someone submits the form

---

### Phase 5: Security & Privacy Considerations

#### Step 13: Implement Security Best Practices

1. **Protect Your Secret Key**:
   - ⚠️ **Note**: With client-side implementation, the secret key is visible to users
   - This provides basic spam protection but is not cryptographically secure
   - For public forms, this is acceptable as the key prevents casual automated abuse
   - For higher security, consider:
     - Implementing Google reCAPTCHA v3
     - Adding server-side validation with a backend proxy
     - Using Google Apps Script's built-in authentication for private forms
   - Rotate the key if you suspect abuse

2. **Restrict Sheet Access**:
   - Share the Google Sheet only with necessary team members
   - Use Google Drive's sharing settings to control access
   - Consider using "View only" access for most users

3. **Data Privacy Compliance**:
   - Ensure compliance with GDPR, HIPAA, or local privacy laws
   - Add a data retention policy
   - Implement data deletion procedures upon request

4. **Rate Limiting** (Advanced):
   - Add rate limiting to the Apps Script to prevent abuse
   - Use Google's built-in quotas and limits

Example rate limiting code (add to `doPost` function):

```javascript
// Simple rate limiting using cache
function checkRateLimit(identifier) {
  const cache = CacheService.getScriptCache();
  const key = 'ratelimit_' + identifier;
  const attempts = cache.get(key);
  
  if (attempts && parseInt(attempts) > 5) {
    return false; // Too many attempts
  }
  
  cache.put(key, (attempts ? parseInt(attempts) + 1 : 1).toString(), 3600); // 1 hour
  return true;
}

// In doPost function, before processing:
if (!checkRateLimit(data.email || data.phone)) {
  return ContentService.createTextOutput(JSON.stringify({
    'status': 'error',
    'message': 'Too many submissions. Please try again later.'
  })).setMimeType(ContentService.MimeType.JSON);
}
```

#### Step 14: Set Up Backups

1. **Enable Version History**:
   - Google Sheets automatically saves version history
   - Access via: File → Version history → See version history

2. **Create Periodic Exports**:
   - Set up a scheduled Apps Script to export data weekly
   - Export to Google Drive or email as CSV

Example backup script:

```javascript
function createWeeklyBackup() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName(SHEET_NAME);
  const folder = DriveApp.getFolderById('YOUR_FOLDER_ID'); // Create a backup folder
  
  const date = Utilities.formatDate(new Date(), 'Asia/Jerusalem', 'yyyy-MM-dd');
  const copy = ss.copy('Waiting List Backup - ' + date);
  
  // Move to backup folder
  const file = DriveApp.getFileById(copy.getId());
  folder.addFile(file);
  DriveApp.getRootFolder().removeFile(file);
}

// Set up a time-based trigger for weekly backups
// In Apps Script: Triggers → Add Trigger → Select createWeeklyBackup → Time-driven → Week timer
```

---

### Phase 6: Advanced Features (Optional)

#### Step 15: Add Data Analysis Dashboard

Create a second sheet called "Dashboard" with:
- Total submissions count
- Submissions by age group (pie chart)
- Submissions by preferred time (bar chart)
- Daily/weekly submission trends
- Average response time metrics

Use formulas like:
```
=COUNTA(Sheet1!A2:A) // Total submissions
=COUNTIF(Sheet1!F2:F,"4-6") // Count of 4-6 age group
=QUERY(Sheet1!A2:L,"SELECT A, COUNT(A) GROUP BY A") // Submissions by date
```

#### Step 16: Automate Follow-Up Workflows

Add an Apps Script trigger to automatically:
- Send follow-up emails after 24 hours
- Mark submissions as "Contacted" or "Scheduled"
- Assign submissions to specific team members

Example:

```javascript
function sendFollowUpEmails() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName(SHEET_NAME);
  const data = sheet.getDataRange().getValues();
  
  const now = new Date();
  const oneDayAgo = new Date(now.getTime() - (24 * 60 * 60 * 1000));
  
  for (let i = 1; i < data.length; i++) { // Start from 1 to skip header
    const timestamp = new Date(data[i][0]);
    const email = data[i][4];
    const contacted = data[i][13]; // Add a "Contacted" column
    
    if (timestamp < oneDayAgo && !contacted && email) {
      // Send follow-up email
      const subject = 'Follow-up: Your Waiting List Submission';
      const body = 'Thank you for joining our waiting list...';
      
      MailApp.sendEmail(email, subject, body);
      
      // Mark as contacted
      sheet.getRange(i + 1, 14).setValue('Yes');
    }
  }
}
```

#### Step 17: Production-Ready Alternatives (For Higher Security Needs)

If you require better error handling, response validation, or enhanced security, consider these alternatives:

**Option A: Backend Proxy with Response Validation**

Create a simple server-side proxy (Node.js, Python, etc.) that:
1. Receives form submissions from your website
2. Validates data server-side
3. Forwards to Google Sheets Web App
4. Returns proper success/error responses to the client

Benefits:
- Full error handling and response validation
- Can hide the secret key server-side
- Better rate limiting and spam protection
- Can add reCAPTCHA verification

**Option B: Google Apps Script HtmlService**

Instead of a public Web App, use Google Apps Script's HtmlService:
1. Create an HTML form using Google Apps Script
2. Embed it in your website via iframe
3. Use `google.script.run` for secure communication
4. Get proper success/failure callbacks

Benefits:
- Built-in authentication if needed
- Better error handling
- No CORS issues
- More secure communication

**Option C: Google Forms Integration**

For simpler use cases:
1. Create a Google Form with matching fields
2. Pre-fill form data via URL parameters
3. Automatically submit or redirect users
4. Responses go directly to Google Sheets

Benefits:
- Zero coding required
- Built-in spam protection
- Official Google integration
- Automatic validation

---

## Implementation Checklist

- [ ] Create Google Sheet with proper columns
- [ ] Set up data validation for dropdown fields
- [ ] Create Apps Script project
- [ ] Copy and configure the Web App script
- [ ] Set secret key in Apps Script
- [ ] Deploy Web App and copy URL
- [ ] Test deployment with `testSubmission()` function
- [ ] Update `index.html` with Google Sheets integration
- [ ] Replace Formspree URL with Google Sheets Web App URL
- [ ] Add secret key to form submission code
- [ ] Test complete form submission flow
- [ ] Verify data appears correctly in Google Sheet
- [ ] Set up email notifications (optional)
- [ ] Implement rate limiting (optional)
- [ ] Create backup strategy
- [ ] Share sheet with team members
- [ ] Document the integration in README.md
- [ ] Update implementation-plan.md

---

## Support & Troubleshooting

### Common Issues

**Issue: "Script function not found: doPost"**
- Solution: Ensure you saved the Apps Script before deploying

**Issue: "Authorization required"**
- Solution: Re-authorize the script: Deploy → Manage deployments → Edit → Re-deploy

**Issue: "Reference Error: SpreadsheetApp is not defined"**
- Solution: You're running the script outside of Google Apps Script environment

**Issue: Data not appearing in correct columns**
- Solution: Check the order of fields in both the form and the `rowData` array

### Getting Help

- **Apps Script Documentation**: https://developers.google.com/apps-script
- **Google Sheets API**: https://developers.google.com/sheets/api
- **Stack Overflow**: Tag questions with `google-apps-script` and `google-sheets`

---

## Additional Resources

- [Google Apps Script Quickstart](https://developers.google.com/apps-script/quickstart/forms)
- [Google Sheets API Documentation](https://developers.google.com/sheets/api/guides/concepts)
- [Web Apps in Apps Script](https://developers.google.com/apps-script/guides/web)
- [Best Practices for Apps Script](https://developers.google.com/apps-script/guides/support/best-practices)

---

**Last Updated**: 2026-01-27
**Issue Status**: Open
**Priority**: High
**Estimated Time**: 2-3 hours for complete implementation
