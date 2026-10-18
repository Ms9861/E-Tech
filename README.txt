# E-Tech Login Website — Setup Guide

This package contains a responsive E-Tech landing/login page and a Google Apps Script backend. The flow is:
1. User enters their User ID/email and password.
2. Backend checks the account against the Google Sheet.
3. Only when credentials match and the account is active, an OTP is emailed to that account's registered email.
4. User enters the OTP to complete sign-in.

## Important setup steps

### 1. Create the Google Sheet
1. Create a new Google Sheet.
2. Copy the ID from its URL: `https://docs.google.com/spreadsheets/d/THIS_PART_IS_THE_ID/edit`
3. Open `Code.gs` in Apps Script (Extensions → Apps Script).
4. Replace `PASTE_GOOGLE_SHEET_ID_HERE` in `CONFIG.SPREADSHEET_ID` with the ID.
5. Save, select `setup` from the function dropdown, and click Run. Authorize the requested Google permissions.

The script creates:
- `Users`: UserID, Email, PasswordHash, Active, CreatedAt
- `LoginLogs`: Timestamp, UserID, Event, Result

### 2. Add an account
In Apps Script, choose `addOrUpdateUser` and click Run. The function requires three arguments, so easiest is to add this temporary helper at the bottom of Code.gs, edit the values, run it once, then delete the helper:

```javascript
function createFirstAccount() {
  addOrUpdateUser("sohail", "your-real-email@example.com", "Use-A-Strong-Password-Here");
}
```

Change the three sample values first. Use a password with at least 10 characters. Run `createFirstAccount`, then delete that helper and save. Repeat with a temporary helper when adding accounts. Do not store plain passwords in the Sheet; the script stores SHA-256 hashes.

### 3. Deploy the backend
1. In Apps Script click **Deploy → New deployment**.
2. Select type **Web app**.
3. Execute as: **Me**.
4. Who has access: **Anyone** (the script itself only exposes the two authentication actions).
5. Deploy and authorize. Copy the Web app URL ending in `/exec`.

### 4. Connect the webpage
Open `index.html` and replace:
`PASTE_YOUR_GOOGLE_APPS_SCRIPT_WEB_APP_URL_HERE`
with the deployed `/exec` URL. Save.

### 5. Publish
Upload `index.html` to your static host (for example, Vercel). Keep `Code.gs` in Apps Script, not in the public website repository if you don't want to expose implementation details.

## Email sender identity
OTP messages are sent by Google Apps Script's Mail service, using **E-Tech** as the display name. Delivery is subject to Google account quotas and mail-provider filtering. The actual sender address is the Google account that owns/runs the script. For a branded address such as `security@yourdomain.com`, the mailbox/domain must be configured with a provider that permits sending as that address; changing the display name alone does not create a verified E-Tech email address.

## Security notes
- Use a private Google Sheet and never share its edit access with public users.
- Keep the deployed Apps Script URL and source configuration under your control.
- OTP expires after 5 minutes; incorrect password attempts are rate-limited and OTP attempts are limited.
- Passwords are not stored in plaintext. SHA-256 hashing here is a lightweight baseline, not a replacement for a dedicated identity provider using a modern password-hashing algorithm. For a public production service, use Firebase Authentication / another managed identity service and server-side session tokens.
- This starter confirms authentication in the browser flow but does not create a long-lived signed session for other protected APIs/pages. Protect any future private data endpoints separately.
