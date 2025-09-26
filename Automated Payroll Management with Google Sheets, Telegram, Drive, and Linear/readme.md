# 📘 n8n Payroll Automation Workflow: Complete Beginner Guide


***

## 1. Introduction

Welcome to your end-to-end guide for the “Google Spreadsheet” n8n workflow project!
This tutorial shows you how to import, configure, and run a payroll automation workflow that logs salary data in Google Sheets, sends Telegram notifications, and updates Linear issues—using exact nodes and settings from your uploaded workflow.

### **Prerequisites:**

- n8n account (self-hosted or cloud)
- Google account (Sheets \& Drive)
- Telegram account (app and bot)
- Linear account (for issue tracking)

***

## 2. How to Import the Workflow in n8n

**Step-by-step:**

1. Log in to n8n.
2. Click **Import** (top or sidebar).
3. Upload your project file: `Google Spreadsheet.json`.
4. The workflow appears on the n8n canvas, auto-populating all node names and settings.

***

## 3. Services \& Nodes Inventory

| Node Name | Provider/Service | Credential Type | Used In (Node Fields) | Custom Code / API / Endpoint |
| :-- | :-- | :-- | :-- | :-- |
| Manual Trigger | n8n | N/A | N/A | N/A |
| Calculate Salaries | n8n Function | N/A | Function node | Uses `item.json["Employee Name"]...` |
| Google Sheets Append | Google Sheets | OAuth2 | Credential dropdown: Google Sheets account 2 | Sheet ID, append operation |
| Send a text message | Telegram | Bot Token | Credential dropdown: Telegram account | chatId, text, parsemode |
| Update file | Google Drive | OAuth2 | Credential dropdown: Google Drive account | FileId, newUpdatedFileName |
| Code in JavaScript | n8n Code | N/A | Custom code node | Reference: `item.json.Employee Name` |
| Update an issue | Linear | OAuth2 | Credential dropdown: Linear account | description, title, issueId |


***

## 4. API Credential Setup \& Service Provisioning

### 4.1 Google Sheets \& Google Drive (OAuth2)

**A. Overview:**
Used for appending payroll data and saving drive files. Requires Google Sheets/Drive API and OAuth2 with Calendar/Drive/Sheets scopes.

**B. Steps to Provision:**

1. Create Google account and login to [Google Cloud Console](https://console.cloud.google.com/).
2. Create a new project.
3. Enable “Google Sheets API” and “Google Drive API” ([Insert screenshot: API Library page]).
4. Go to “APIs \& Services → Credentials → Create credentials → OAuth client ID” ([Insert screenshot: Credentials creation]).
5. Set “Application type”: Web App.
6. Redirect URI (n8n):
    - For n8n cloud: `https://app.n8n.cloud/rest/oauth2-credential/callback`
    - For self-hosted: `http://localhost:5678/rest/oauth2-credential/callback` or your deployed domain.
7. Give consent screen info and click “Create”.
8. Copy `CLIENT_ID` and `CLIENT_SECRET` ([Insert screenshot: OAuth creds page]).
9. Save these for n8n set-up.

**C. n8n Credential Setup:**

- Go to Credentials > New > Google Sheets/Google Drive OAuth2.
- Paste Client ID, Client Secret.
- Redirect URI: copy from n8n credential setup screen.
- Test and save.

**D. Example Node Setup:**

- Node: **Google Sheets Append**
    - Select credential: Google Sheets account 2
    - Sheet ID: `1x6rwWFWh7LkIkCl9FI-lZju8yrs4j3WIbikdjGiM4w`
    - Value Input Mode: RAW
- Node: **Update file** (Google Drive)
    - Select credential: Google Drive account
    - FileId: Use file URL for your sheet
    - newUpdatedFileName: `Payroll-Backup-{{$now.format('dd-LL-yyyy')}}.xlsx`

**E. Test \& Verify:**

- Open Google Sheets/Drive node in n8n, click "Test Credential" ([Insert screenshot: Test button]).
- Should return OK with 200, file and sheet access confirmed.
- For errors: check scope, redirect URI, and client type.

***

### 4.2 Telegram (Bot Token)

**A. Overview:**
Telegram node sends payroll notifications. Needs Telegram bot token and chat/user ID.

**B. Steps to Provision:**

1. Open Telegram app.
2. Search @BotFather, start chat ([Insert screenshot: BotFather chat]).
3. Send `/newbot`, follow prompts (name and username).
4. Copy returned Bot Token.
5. (Optional) Set description, photo: `/setdescription`, `/setuserpic`.
6. Send `/setprivacy` to toggle privacy if using in group.
7. Paste token in n8n credential.

**C. n8n Credential Setup:**

- Go to Credentials > New > Telegram API.
- Paste token: `BOT_TOKEN = <123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11>`

**D. Example Node Setup:**

- Node: **Send a text message**
    - Credential: Telegram account
    - chatId: Paste numeric chat ID
    - text: Use expressions like `Payroll entry for {{$json["Employee Name"]}}`
    - parsemode: HTML

**E. Test \& Verify:**

- Execute node manually in n8n.
- Expect a Telegram message in selected chat.
- Errors: 403 (bot not started/chat), 400 (bad chat ID), 401 (invalid token).

***

### 4.3 Linear (OAuth2)

**A. Overview:**
Manages issue tracking, updates payroll info to Linear as issues.

**B. Steps to Provision:**

1. Login to Linear ([https://linear.app](https://linear.app)).
2. In Settings > API > Create new OAuth Application ([Insert screenshot: Linear API settings]).
3. Set App name: e.g. “n8n Payroll”.
4. Redirect URI: from n8n credential screen.
5. Choose scopes: “Issues:write”, “Issues:read”.
6. Save and copy `CLIENT_ID` / `CLIENT_SECRET`.

**C. n8n Credential Setup:**

- Go to Credentials > New > Linear OAuth2.
- Paste client ID and secret.
- Set redirect URI as before.

**D. Example Node Setup:**

- Node: **Update an issue**
    - Credential: Linear account
    - issueId: your Linear Issue ID (shown in Linear web app)
    - assigneeId: user/group ID
    - description: Use `{{$json.description}}` (from previous Code node)
    - title: PAYROLL

**E. Test \& Verify:**

- Execute node to update issue in Linear.
- Confirm issue is updated via app; see output in n8n.

***

## 5. Telegram Bot \& Chat ID Quick Reference

### A. Create Telegram Bot

1. In Telegram app, search @BotFather, `/newbot`.
2. Give bot name/username, copy token.
3. Use `/setprivacy` as needed.
4. Read usage notes: bots cannot message a user/group without “starting” first.

### B. Get Telegram Chat/User ID

**Method 1: @userinfobot**

- Search @userinfobot, send `/start`.
- Receives reply: “Your Telegram ID: 123456789”

**Method 2: Get Updates API**

- Send message to your bot or group.
- Use:

```bash
curl -s "https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates" | jq
```

- Find `"chat":{"id":...}` in output. Group chat IDs are negative.

**Method 3: Capture with n8n**

- Use Telegram Trigger node, check message payload for `chat.id`.


### C. Configure Telegram Node

- Bot Token: paste from BotFather.
- Chat ID: paste Numeric ID (e.g. `2058407139` or group ID as `-123456789`)
- Text: `Payroll entry for {{$json["Employee Name"]}}...`
- parsemode: HTML/Markdown


### D. Test Integration

- Run n8n node.
- Confirm message arrives in Telegram chat.


### Troubleshooting

- 403: add bot to group/user must start chat.
- 400: verify chat ID and bot token.
- 401: confirm token/correct bot.

***

## 6. Node Configuration Walkthrough

**Manual Trigger**: Start workflow manually (no setup).

**Calculate Salaries**: Built-in sample data, function formats salary/date:

```js
item.json["Employee Name"] // Use for downstream nodes
```

Outputs array of records like:

```json
{ "Employee Name": "Vignesh", "Designation": "Web Developer", "Salary": 12000, "Date": "25/09/2025" }
```

**Google Sheets Append**:

- Credential: Google Sheets account 2
- Sheet ID: Use your Google Sheet ID.
- Values: `{{$json["Employee Name"]}}`, `{{$json["Salary"]}}`, etc.

**Send a text message**:

- Credential: Telegram account
- chatId: 2058407139 (example)
- text: `Payroll entry for {{$json["Employee Name"]}} ({{$json["Designation"]}}) of ₹{{$json["Salary"]}}...`

**Update file**:

- Credential: Google Drive account
- fileId: (your spreadsheet URL)
- newUpdatedFileName: Payroll-Backup-date.xlsx

**Code in JavaScript**:

- Inputs: Builds Markdown table from payroll JSON objects.
- Outputs: `json.description`—used downstream for Linear node.

**Update an issue**:

- Credential: Linear account
- issueId: (Linear issue ID, e.g., WEB-108)
- updateFields: assigneeId, description (Markdown table), title.

***

## 7. Run \& Test the Workflow

1. Click **Execute Workflow** on Manual Trigger node.
2. Review execution steps and logs inside n8n.
3. Check Google Sheet, Telegram app, Google Drive, and Linear issues for updates.

***

## 8. Common Errors \& Troubleshooting

| Error | How to Fix |
| :-- | :-- |
| `redirect_uri_mismatch` | Ensure redirect URI matches n8n credentials exactly. |
| `invalid_client` | Re-copy and paste Client ID and Secret. |
| `unauthorized` (401/403) | Confirm token, chat/user ID, API/key setup. |
| `rate_limit` | Wait and retry, check quotas in cloud/API dashboard. |
| `missing scopes` | Edit API/app config to add needed scopes and re-authenticate. |
| Telegram: 403 Forbidden | Add bot to group; user must start chat with bot. |
| Telegram: 400 Bad Request | Check chat ID used—should be numeric and match intended chat. |


***

## 9. Security Best Practices

- **Minimal scopes:** Only enable required API scopes.
- **Key rotation:** Regularly update credentials, revoke unused keys.
- **Restrict API key usage:** By IP, referrer, or app.
- Never share or commit secret keys/tokens to repos.
- Use environment variables for secrets (.env)
- Remove API keys from workflows shared publicly.

***

## 10. Maintenance \& Token Rotation

- Track expiration/rotation for OAuth2 tokens (Google, Linear).
- Set up reminders for periodic credential review.
- Revoke old credentials in cloud console/providers.

***

## 11. Appendices

**Sample cURL - Telegram sendMessage:**

```bash
curl -X POST "https://api.telegram.org/bot<YOUR_BOT_TOKEN>/sendMessage" \
  -d chat_id=<CHAT_ID> \
  -d text="Test message from n8n"
```

**Useful console links:**

- [Google Cloud Console](https://console.cloud.google.com/)
- [Telegram BotFather](https://t.me/BotFather)
- [Linear App](https://linear.app)

**[Insert screenshot: each credential and node config page]**

***

## 12. Deliverables

### A. Credentials Cheat-Sheet

| Provider | Credential Type | Paste Location (n8n) | Redirect URI (if needed) |
| :-- | :-- | :-- | :-- |
| Google Sheets | OAuth2 | Google Sheets OAuth Credential | https://app.n8n.cloud/rest/oauth2-credential/callback |
| Google Drive | OAuth2 | Google Drive OAuth Credential | https://app.n8n.cloud/rest/oauth2-credential/callback |
| Telegram | Bot Token | Telegram API Credential | N/A |
| Linear | OAuth2 | Linear OAuth Credential | https://app.n8n.cloud/rest/oauth2-credential/callback |


***

### B. README Snippet

```
# n8n Payroll Automation Workflow

Import this workflow in n8n, set up your Google Sheets/Drive, Telegram, and Linear API credentials, and run the workflow to automatically log payroll entries, send notifications, save backups, and manage issues!
See the full tutorial for setup steps, credential creation, and troubleshooting.
```


***

### C. Printable One-Page Setup Checklist

**Setup Checklist**

- [ ] Import workflow file in n8n
- [ ] Create/connect Google Cloud Project (Sheets, Drive APIs enabled)
- [ ] Get client ID/secret for OAuth2
- [ ] Create/connect Telegram Bot (BotFather, get token)
- [ ] Obtain Telegram chat/user ID
- [ ] Create/connect Linear OAuth2 app \& obtain credentials
- [ ] Add credentials for all above in n8n
- [ ] Map fields and check values in nodes (Sheet ID, chat ID)
- [ ] Execute workflow, confirm updates in Sheet, Telegram, Drive, Linear
- [ ] Verify security and rotate tokens periodically

***

### D. Telegram Quick Reference

- Create bot with BotFather, copy token.
- Get chat ID using @userinfobot or /getUpdates.
- Add bot to group if needed.
- Configure Telegram node with credentials and IDs.
- Test sendMessage with cURL if troubleshooting.

***

## 13. Security \& Verification

- Follow official provider guides: [Google Sheets OAuth](https://developers.google.com/identity/protocols/oauth2), [Telegram Bots API](https://core.telegram.org/bots), [Linear API Docs](https://developers.linear.app/docs/)
- If you encounter “label not found,” always use provider search bar and documentation links.
- Use n8n’s Test button for credential validation.

***

**Done! If you need a visual PDF, a sample environment file, or require advanced customizations (e.g., more notifications, batch reporting), let me know! This tutorial covers every step to fully implement and securely run the workflow with all integrations.**
<span style="display:none">[^1]</span>

```
<div style="text-align: center">⁂</div>
```

[^1]: Google-Spreadsheet.json

