# MR. QR Scanner (2026 Edition)

A battery-optimized, high-performance QR code scanner built for office environments, attendance tracking, and data logs. This app seamlessly records **TIME IN** and **TIME OUT** scans directly into separate tabs of a designated Google Sheet via background workers, preventing data loss and eliminating duplicated sync entries.

## 📥 Download App

Click the link below to get the latest stable, battery-optimized version:

* **[Download MR. QR Scanner APK](https://github.com/mwsr-max/MR-QR-Scanner/releases/download/v1.0/MR_QR_Scanner_v1.0.apk)**

---

## 🛠️ Features

* **Smart Log Validation:** Prevents accidental double-scanning by checking the database history in real-time. If a user attempts to scan the exact same QR code twice within the same day, the second scan is blocked, and the app instantly informs them with a clear toast notification: `"ALREADY LOGGED TIME IN/OUT TODAY"`.
* **Camera Power Management:** Built-in manual shutdown button and automatic lifecycle tracking to disable the camera when changing tabs, saving crucial battery life.
* **Local History Log:** Dark-mode text optimized, high-contrast, scrollable history list with an option to clear all entries or export to a local CSV file.
* **Seamless Google Sheets Integration:** Automatically routes scan records to matching sheets/tabs ("TIME IN" or "TIME OUT") dynamically.

---

## 📊 Google Sheets Setup Tutorial

To connect the app to your own database, you must deploy a Google Apps Script web app. Follow these steps:

### 1. Create Your Spreadsheet
1. Open [Google Sheets](https://sheets.google.com) and create a new blank spreadsheet.
2. Save it with a name like `MR. QR Scanner Attendance`.

### 2. Add the Web App Script
1. In the top menu of your Google Sheet, go to **Extensions** > **Apps Script**.
2. Delete any default code in the editor and paste the following script:

```javascript
function doPost(e) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var data = JSON.parse(e.postData.contents);
  
  // Use the 'type' from the app (TIME IN / TIME OUT) to find the right sheet tab
  var sheetName = data.type; 
  var sheet = ss.getSheetByName(sheetName);
  
  // Create the sheet tab automatically if it doesn't exist
  if (!sheet) {
    sheet = ss.insertSheet(sheetName);
    sheet.appendRow(["QR Data", "Date Log", "Time Log", "User ID"]);
  }
  
  try {
    sheet.appendRow([
      data.qr_content,
      data.date,
      data.time,
      data.user_name
    ]);
    return ContentService.createTextOutput(JSON.stringify({"status": "success"}))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService.createTextOutput(JSON.stringify({"status": "error", "message": err.toString()}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```
### 3. Deploy the Web App
1. Click the **Save** icon (floppy disk) at the top of the Apps Script editor.
2. Click the blue **Deploy** button > **New deployment**.
3. Click the gear icon next to "Select type" and select **Web app**.
4. Configure the deployment parameters exactly like this:
   * **Description:** `MR. QR Scanner Sync API`
   * **Execute as:** `Me (your-email@gmail.com)`
   * **Who has access:** `Anyone` *(Crucial: This lets your mobile app safely send logs in the background).*
5. Click **Deploy**.
6. Google will prompt you to authorize permissions. Click **Authorize Access**, choose your Google account, click **Advanced** at the bottom, and click **Go to Untitled project (unsafe)** to grant the required script access.
7. Once successfully deployed, a window will display a **Web app URL**. Copy this URL link (it should end in `/exec`).

### 4. Link the URL to Your App Settings
To activate the cloud synchronization feature, you must save your deployment link inside the application's configuration dashboard:

1. Open **MR. QR Scanner** on your Android device.
2. Navigate over to the **Settings** tab via the bottom navigation bar.
3. Paste your copied Google Web App URL into the text box labeled **Google Sheets URL**.
4. (Optional) Provide your active tracking profile ID in the **Username/User ID** section if required.
5. Tap the **SAVE SETTINGS** button.

The app will instantly lock this URL configuration in place. Background sync workers will now automatically route your time log metrics straight to your specific spreadsheet file whenever an active internet connection is detected!

---

## 🔒 Permissions Used

* `android.permission.CAMERA` — For streaming the video feed to scan barcodes.
* `android.permission.VIBRATE` — For supplying physical haptic feedback upon logging scans.
* `android.permission.INTERNET` — Used to execute network synchronization back to Google Sheets.
