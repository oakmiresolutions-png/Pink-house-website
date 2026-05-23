# Google Sheets Form Endpoint

Status: blocked until the Google Apps Script API is enabled for OAuth project `905799050117`.

Sheet:
https://docs.google.com/spreadsheets/d/1vK0_1FUTH-lsY9p6TTE166o1YRrn_iK2K3rUX0TJkPU/edit

Required columns already added to `Sheet1!A1:I1`:
- Timestamp
- Full Name
- Phone
- City / Service Address
- Property Type
- Urgency
- Service Needed
- Description
- Source

API to enable:
https://console.developers.google.com/apis/api/script.googleapis.com/overview?project=905799050117

Apps Script code for the endpoint:

```javascript
const SHEET_ID = '1vK0_1FUTH-lsY9p6TTE166o1YRrn_iK2K3rUX0TJkPU';
const SHEET_NAME = 'Sheet1';

function doGet() {
  return ContentService
    .createTextOutput('Pink House lead endpoint is active.')
    .setMimeType(ContentService.MimeType.TEXT);
}

function doPost(e) {
  const lock = LockService.getScriptLock();
  lock.waitLock(10000);

  try {
    const data = readPayload_(e);
    const sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName(SHEET_NAME);
    sheet.appendRow([
      new Date(),
      clean_(data.name),
      clean_(data.phone),
      clean_(data.location),
      clean_(data.property_type),
      clean_(data.urgency),
      clean_(data.service),
      clean_(data.description),
      clean_(data.source || 'Website')
    ]);

    return json_({ ok: true });
  } catch (error) {
    return json_({ ok: false, error: String(error && error.message ? error.message : error) });
  } finally {
    lock.releaseLock();
  }
}

function readPayload_(e) {
  if (e && e.parameter && Object.keys(e.parameter).length) {
    return e.parameter;
  }

  const contents = e && e.postData ? e.postData.contents : '';
  if (!contents) return {};

  try {
    return JSON.parse(contents);
  } catch (error) {
    return contents.split('&').reduce(function(values, pair) {
      const parts = pair.split('=');
      values[decodeURIComponent(parts[0] || '')] = decodeURIComponent((parts[1] || '').replace(/\+/g, ' '));
      return values;
    }, {});
  }
}

function clean_(value) {
  return String(value || '').trim();
}

function json_(payload) {
  return ContentService
    .createTextOutput(JSON.stringify(payload))
    .setMimeType(ContentService.MimeType.JSON);
}
```

Once the web app URL exists, update the website form to post to it and test with a fake lead.
