# Wiring the survey to a real Google Sheet — exact steps

This makes every survey submission actually save to a Google Sheet, and makes the trainer dashboard show real counts instead of sample data. Takes about 5 minutes, one-time.

## 1. Create the Sheet

1. Go to [sheets.google.com](https://sheets.google.com), create a new blank sheet.
2. Rename it something like `WGP Workshop Responses`.
3. In row 1, add these exact headers (one per column, A through J):
   `Timestamp | Type | Name | Phone | Industry | TeamSize | Challenge | Goal | MemberID | StepsDone`
   (The `Type` column separates survey responses from homework "I'm ready" submissions; `StepsDone` records how far someone got in the homework.)

## 2. Add the script that receives submissions

1. In the Sheet, go to **Extensions → Apps Script**.
2. Delete whatever's in the editor, paste this in exactly:

```javascript
function doPost(e) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const d = JSON.parse(e.postData.contents);
  sheet.appendRow([
    new Date(),
    d.type || "survey",
    d.name || "",
    d.phone || "",
    d.industry || "",
    d.team || "",
    d.challenge || "",
    d.goal || "",
    d.memberId || "",
    d.stepsDone || ""
  ]);
  return ContentService.createTextOutput(JSON.stringify({ status: "ok" }))
    .setMimeType(ContentService.MimeType.JSON);
}

function doGet(e) {
  // Lets the trainer dashboard read live counts back out, as JSON.
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const rows = sheet.getDataRange().getValues();
  const headers = rows.shift();
  const data = rows.map(r => {
    const obj = {};
    headers.forEach((h, i) => obj[h] = r[i]);
    return obj;
  });
  return ContentService.createTextOutput(JSON.stringify(data))
    .setMimeType(ContentService.MimeType.JSON);
}
```

3. Click **Save** (disk icon), name the project `WGP Survey Handler`.

## 3. Deploy it as a Web App

1. Click **Deploy → New deployment**.
2. Click the gear icon next to "Select type" → choose **Web app**.
3. Set:
   - **Execute as:** Me (your account)
   - **Who has access:** Anyone
4. Click **Deploy**. Google will ask you to authorize — click through (it's your own script, this is expected).
5. Copy the **Web app URL** it gives you — looks like `https://script.google.com/macros/s/AKfycb.../exec`.

## 4. Give me that URL

Send me the Web app URL and I'll drop it into both `survey.html` (so submissions save there) and `trainer-dashboard.html` (so it reads real counts back). Both files already have a config slot ready for it — see `SHEET_WEBHOOK_URL` near the top of each script.

## Note on re-deploying

If you ever edit the Apps Script code later, you must do **Deploy → Manage deployments → edit (pencil) → New version** — saving the script alone does NOT push changes to the live URL.
