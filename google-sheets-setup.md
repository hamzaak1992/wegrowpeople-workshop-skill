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

Surveys land on your first tab; homework "I'm ready" submissions land on a separate **Homework** tab (auto-created on the first submission).

**Anyone can submit (that's the point), but only the team can read the data back.** Reading requires the password below. Without it, the sheet's web address returns nothing useful even to someone who finds it in the page source.

```javascript
// Password required to READ the data. Change it here and in the dashboard together.
var READ_KEY = 'wgP2026';

var HW_HEADERS = ['Timestamp','Name','Phone','Steps Done'];

function doPost(e) {
  var d = JSON.parse(e.postData.contents);
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var isHomework = String(d.type).toLowerCase().indexOf('homework') >= 0;
  if (isHomework) {
    var hw = ss.getSheetByName('Homework');
    if (!hw) { hw = ss.insertSheet('Homework'); hw.appendRow(HW_HEADERS); }
    hw.appendRow([ new Date(), d.name || "", d.phone || "", d.stepsDone || "" ]);
  } else {
    var sv = ss.getSheets()[0]; // your first tab = surveys
    sv.appendRow([
      new Date(), d.type || "survey", d.name || "", d.phone || "",
      d.industry || "", d.team || "", d.challenge || "", d.goal || "",
      d.memberId || "", d.stepsDone || ""
    ]);
  }
  return ContentService.createTextOutput(JSON.stringify({ status: "ok" }))
    .setMimeType(ContentService.MimeType.JSON);
}

function readSheet_(sheet) {
  if (!sheet) return [];
  var rows = sheet.getDataRange().getValues();
  if (rows.length < 2) return [];
  var headers = rows.shift();
  return rows.map(function(r){ var o={}; headers.forEach(function(h,i){ o[h]=r[i]; }); return o; });
}

function doGet(e) {
  // Locked: without the right key this returns nothing, so attendee names,
  // phone numbers and answers aren't readable by anyone who finds this URL.
  var key = (e && e.parameter && e.parameter.key) || '';
  if (key !== READ_KEY) {
    return ContentService.createTextOutput(JSON.stringify({ error: 'unauthorized' }))
      .setMimeType(ContentService.MimeType.JSON);
  }
  // Returns both tabs so the trainer dashboard can show surveys and set-up counts separately.
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var out = {
    surveys: readSheet_(ss.getSheets()[0]),
    homework: readSheet_(ss.getSheetByName('Homework'))
  };
  return ContentService.createTextOutput(JSON.stringify(out))
    .setMimeType(ContentService.MimeType.JSON);
}
```

**If you already deployed the earlier single-tab version:** just replace the code with the above, then re-deploy the SAME deployment (**Deploy → Manage deployments → ✏️ Edit → Version: New version → Deploy**) so the URL stays the same. The Homework tab appears automatically the first time someone submits.

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
