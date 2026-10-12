# Nirosha & Tharusha — Wedding Invitation

A self-contained, bilingual (English + Sinhala) wedding invitation website.
Burgundy & beige theme with gold accents, ambient music, animated petals,
an agenda, a save-the-date calendar, embedded maps, and a Google-Sheet–backed RSVP.

Everything (styles, content, logic) lives in a single file: **`index.html`**.

---

## 1. Deploy for free (GitHub Pages)

The repo is already on GitHub, so the easiest free host is GitHub Pages.

1. Push `index.html` to your `main` branch.
2. On GitHub, open the repo → **Settings → Pages**.
3. Under **Build and deployment**:
   - **Source**: `Deploy from a branch`
   - **Branch**: `main` → `/ (root)`
4. Save. After ~1 minute your site is live at:

```
https://<your-username>.github.io/my_wedding/
```

> (If your repo has a different name, the URL uses that name.)

---

## 2. RSVP → Google Sheet (one-time setup)

The RSVP form posts each reply to a Google Sheet via a Google Apps Script Web App.
Responses are stored as rows so you can view them in the sheet.

### Step A — Create the sheet
1. Go to [sheets.new](https://sheets.new) and name it e.g. `Wedding RSVP`.

### Step B — Add the script
1. In the sheet, click **Extensions → Apps Script**.
2. Delete any sample code and paste the script below:

```javascript
function doPost(e) {
  var lock = LockService.getScriptLock();
  lock.tryLock(10000);
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
    var data = {};
    try { data = JSON.parse(e.postData.contents); } catch (err) {}

    if (sheet.getLastRow() === 0) {
      sheet.appendRow(['Timestamp', 'Name', 'Attending', 'Message', 'Language', 'Source', 'Submitted At']);
    }

    sheet.appendRow([
      new Date(),
      data.name || '',
      data.attending || '',
      data.message || '',
      data.lang || '',
      data.source || '',
      data.at || ''
    ]);

    return ContentService
      .createTextOutput(JSON.stringify({ ok: true }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ ok: false, error: String(err) }))
      .setMimeType(ContentService.MimeType.JSON);
  } finally {
    lock.releaseLock();
  }
}
```

3. Click **Save** (the disk icon).

### Step C — Deploy as a Web App
1. Click **Deploy → New deployment**.
2. Click the gear icon and choose **Web app**.
3. Set:
   - **Execute as**: `Me`
   - **Who has access**: `Anyone`
4. Click **Deploy**, then **Authorize access** when prompted.
5. Copy the **Web app URL** (it ends in `/exec`).

### Step D — Connect the website
1. Open `index.html` and find the RSVP config in the `invite-data` JSON block:
   ```json
   "rsvp": {
     "line": { ... },
     "deadlineISO": "2026-10-31T23:59:59+05:30",
     "contacts": [ ... ],
     "endpoint": ""
   }
   ```
2. Paste your Web app URL into `"endpoint"`:
   ```json
   "endpoint": "https://script.google.com/macros/s/XXXXXXXX/exec"
   ```
3. Save, commit, and push.

> Until you paste the endpoint, the **"Send RSVP"** button gracefully falls back
> to sending the reply via WhatsApp (so nothing breaks while you're setting it up).

---

## 3. Customize the content

Everything editable lives in the `<script type="application/json" id="invite-data">`
block near the top of `index.html`:

- **`couple`** — bride & groom names (English + Sinhala)
- **`date`** — the wedding date
- **`events`** — the agenda timeline (time, venue, area, start/end)
- **`venues`** — the two locations, including their `mapEmbed` Google Maps URLs
- **`rsvp`** — reply deadline, WhatsApp/SMS contacts, and the sheet `endpoint`
- **`ui`** — all on-page labels (English + Sinhala)

Change a value, save, and the whole page updates (both languages) automatically.

---

## 4. Local preview

Just double-click `index.html` to open it in a browser — no build step or server needed.
