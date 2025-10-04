# Automate Employee Attendance Tracking with WhatsApp, Google Sheets, Drive, and Telegram using n8n


## What You’ll Learn
- **WhatsApp Integration:** Capture and automate incoming attendance messages.
- **Google Sheets Automation:** Log each IN/OUT with accurate timestamps in a spreadsheet.
- **Google Drive Sync:** Automatically update/export your attendance file for backup or distribution.
- **Telegram Notifications:** Instantly confirm entries to HR/admins via chat.
- **JavaScript Parsing:** Use a Code node to extract and format message details inside n8n.
- **End-to-End Automation:** Combine all tools into one seamless, no-code workflow.

## Project Purpose
Track employee IN/OUT times sent via WhatsApp:
- Example: “Kumar OUT 7:30pm”
- Record every entry in Google Sheets with time and message details
- Instantly update/export the attendance log to Drive
- Send a confirmation message to a Telegram group


## Step-by-Step Setup Instructions
### 1. Access or Install n8n

- **Cloud:** Sign up at [n8n.cloud](https://n8n.cloud/)
- **Local (Docker):**

```bash
docker run -it --rm -p 5678:5678 n8nio/n8n
```

- Open the n8n editor UI in your browser.

### 2. Create a New Workflow

- Click `+ New Workflow`
- Name it: **Attendance Tracker**

### 3. Add WhatsApp Trigger Node

- Add a **WhatsApp Trigger** node.
- Connect it to your WhatsApp Business API or preferred provider (Twilio, WATI, etc.).
- Configure OAuth/API credentials in `WhatsApp OAuth account`.
- Set up webhook in your provider to point to the node’s webhook URL.

### 4. Add a Code (JavaScript) Node

- Drag in a **Code** node; name it “Parse WhatsApp Message”.
- Connect it after WhatsApp Trigger.
- Use this code to extract name, IN/OUT status, time, and date:

```javascript
// Only allow processing between 8:00–20:00 IST
const now = new Date();
const hoursIST = now.toLocaleString('en-US', { hour: '2-digit', hour12: false, timeZone: 'Asia/Kolkata' });
if (parseInt(hoursIST, 10) < 8 || parseInt(hoursIST, 10) > 20) return [];

// --- Parse WhatsApp payload ---
const data = items[^0].json;
const userMessage = data.messages?.[^0]?.text?.body;
const name = data.contacts?.[^0]?.profile?.name || (userMessage || '').split(" ")[^0] || "Unknown Employee";
const messageText = (userMessage || "").toLowerCase();
const status = messageText.includes("out") ? "OUT" : "IN";
const regTime = (userMessage.match(/\d{1,2}:\d{2}[ap]m/i) || [])[^0] || "Time Not Found";
const timestamp = new Date(parseInt(data.messages?.[^0]?.timestamp) * 1000);

const timeOptions = { hour: '2-digit', minute: '2-digit', hour12: true, timeZone: 'Asia/Kolkata' };
let time = timestamp.toLocaleTimeString('en-IN', timeOptions);
time = time.replace(/([ap]m)/i, (match) => match.toUpperCase());

const dateOptions = { day: '2-digit', month: '2-digit', year: 'numeric', timeZone: 'Asia/Kolkata' };
const date = timestamp.toLocaleDateString('en-GB', dateOptions);

return [{
  json: { Name: name, Status: status, Time: time, Date: date, UserMessage: userMessage, rawTimestamp: data.messages?.[^0]?.timestamp }
}];
```

### 5. Add Google Sheets Node

- Add **Google Sheets** node after the Code node.
- Set operation to **Append Row**.
- Connect Google Sheets OAuth2 credentials.
- **Paste your Sheet ID** (from the URL) and Sheet Name (e.g., “Sheet1”).
- **Auto-map/Manually map columns:**

| Google Sheet Column | Data Mapped From Node |
| :------------------ | :-------------------- |
| Name                | `Name`                |
| Status              | `Status`              |
| Time                | `Time`                |
| Date                | `Date`                |
| UserMessage         | `UserMessage`         |
| rawTimestamp        | `rawTimestamp`        |

- Format Sheet columns with headers for readability.

### 6. Add Google Drive Node

- Add **Google Drive** node next.
- Set operation to **Update File** (using your Sheet’s file ID) for interactive updates, or **Export File** to create a distribution-ready version.
- Connect Google Drive OAuth2 credentials.
- **Paste your File ID** (from Google Drive file URL).

### 7. Add Telegram Node

- Add **Telegram** node at the end of the flow.
- Connect with your Bot token (created via @BotFather).
- Set your target chat ID.
- Set Text to:

```markdown
{{ $json["Name"] }} ({{ $json["Status"] }} at {{ $json["Time"] }} on {{ $json["Date"] }})
```

    - Set Parse Mode: `HTML` or `Markdown` (per your preference)

## Google Sheet Formatting

- Add columns in **Row 1**:
  `Name | Status | Time | Date | UserMessage | rawTimestamp`
- Freeze the first row for clarity.
- Format `Date` as `DD/MM/YYYY`, and `Time` as `hh:mm AM/PM` for dashboards.

## Execution Flow: Example

**Sample WhatsApp Message:**
`Kumar OUT 7:30pm`

- **Trigger:** WhatsApp Trigger node fires on receiving the message.
- **Parse:** Code node extracts
  - Name: “Kumar”
  - Status: “OUT”
  - Time: “07:30 PM”
  - Date: Today’s date (auto-formatted IST)
  - UserMessage: original text
  - rawTimestamp: exact WhatsApp timestamp
- **Sheets:** Node appends a new row with all data above.
- **Drive:** Node updates or exports the attendance file for backup/distribution.
- **Telegram:** Bot sends “Kumar (OUT at 07:30 PM on 26/09/2025)” to your group/channel.

## How to Test

1. **Send Test WhatsApp Message:**
   Message your WhatsApp business number with “Testuser IN 8:55am”.
2. **Check Google Sheet:**
   - Confirm a new row is added in real-time.
3. **Check Google Drive:**
   - Updated/exported file will appear in your drive or shared folder.
4. **Check Telegram:**
   - You (or your HR group) should receive instant confirmation.
5. **Debugging:**
   - If any node fails, check execution logs in n8n (shows error, payload, and suggestions).

## Error Handling Tips

- **Validate message format:**
  In the Code node, return an error or skip processing if the message is malformed.
- **Enable `errorWorkflow`:**
  - In workflow settings, point `errorWorkflow` to another workflow that logs or alerts on errors.
- **Google API Errors:**
  - Ensure your OAuth credentials have correct permissions and files are shared appropriately.

## Mini-Challenge: Level Up!

**Enhance this workflow:**

- Add another Code and Sheets node to generate a daily summary (“3 employees present, 2 absent on 26/09/2025”).
- Send the summary at 8pm every day to Telegram or your HR’s email using the Email node.
- Try adding Slack or Discord notifications!

**Congratulations!**
You’ve built a fully automated employee attendance system powered by WhatsApp, Google, and Telegram—all using n8n and zero code deployments.
Refine the workflow as your team grows and needs evolve!
