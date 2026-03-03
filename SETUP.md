# Setup Guide - Form to Multi-Channel Notifications

This guide walks you through the complete setup process for deploying the workflow.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [n8n Installation](#n8n-installation)
3. [Credential Configuration](#credential-configuration)
4. [Workflow Import](#workflow-import)
5. [Testing](#testing)
6. [Production Deployment](#production-deployment)

## Prerequisites

### Required Accounts

- [ ] n8n instance (self-hosted or cloud)
- [ ] Gmail account with 2FA enabled
- [ ] Telegram account
- [ ] Google account (for Sheets)

### Required Tools

- cURL or Postman (for testing)
- Text editor
- Web browser

## n8n Installation

### Cloud Option (Recommended for Beginners)

1. Visit [n8n.cloud](https://n8n.cloud/)
2. Sign up for a free account
3. Create a new workspace
4. Skip to [Credential Configuration](#credential-configuration)

### Self-Hosted Option

#### Using Docker (Recommended)

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

#### Using npm

```bash
npm install n8n -g
n8n start
```

Access n8n at `http://localhost:5678`

## Credential Configuration

### 1. Gmail OAuth2 Setup

#### Step 1: Create Google Cloud Project

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Click **Create Project**
3. Name: "n8n Integration"
4. Click **Create**

#### Step 2: Enable Gmail API

1. Navigate to **APIs & Services** → **Library**
2. Search for "Gmail API"
3. Click **Enable**

#### Step 3: Create OAuth Credentials

1. Go to **APIs & Services** → **Credentials**
2. Click **+ CREATE CREDENTIALS** → **OAuth client ID**
3. Application type: **Web application**
4. Name: "n8n Gmail"
5. Authorized redirect URIs:
   - `https://your-n8n-domain.com/rest/oauth2-credential/callback`
6. Click **Create**
7. Copy **Client ID** and **Client Secret**

#### Step 4: Configure in n8n

1. In n8n, go to **Credentials** → **New**
2. Select **Gmail OAuth2 API**
3. Paste Client ID and Client Secret
4. Click **Connect my account**
5. Authorize access
6. Save credential as "Gmail account"

### 2. Telegram Bot Setup

#### Step 1: Create Bot

1. Open Telegram and search for [@BotFather](https://t.me/botfather)
2. Send `/newbot`
3. Choose a name: "Form Notifications Bot"
4. Choose a username: "your_form_bot"
5. Copy the **bot token** (format: `1234567890:ABCdefGHIjklMNOpqrsTUVwxyz`)

#### Step 2: Get Chat ID

1. Search for [@userinfobot](https://t.me/userinfobot)
2. Start a conversation
3. Copy your **Chat ID** (numeric value)

#### Step 3: Configure in n8n

1. In n8n, go to **Credentials** → **New**
2. Select **Telegram API**
3. Paste your bot token
4. Save credential as "Telegram account"

### 3. Google Sheets Setup

#### Step 1: Create Spreadsheet

1. Go to [Google Sheets](https://sheets.google.com/)
2. Create a new spreadsheet
3. Name it: "Form Submissions"
4. Add headers in row 1:
   - A1: Name
   - B1: Email
   - C1: Phone
   - D1: Company
   - E1: Service
   - F1: Timestamp
   - G1: Source
   - H1: Status

#### Step 2: Get Spreadsheet ID

From the URL:
```
https://docs.google.com/spreadsheets/d/[SPREADSHEET_ID]/edit
```
Copy the `SPREADSHEET_ID` portion

#### Step 3: Configure OAuth2

1. Use the same Google Cloud Project from Gmail setup
2. Enable **Google Sheets API** in the API Library
3. Use the same OAuth credentials

#### Step 4: Configure in n8n

1. In n8n, go to **Credentials** → **New**
2. Select **Google Sheets OAuth2 API**
3. Use existing OAuth credentials
4. Authorize access
5. Save credential as "Google Sheets account"

## Workflow Import

### Step 1: Download Workflow

Download `Form-to-Multi-Channel-Notifications.json` from this repository

### Step 2: Import to n8n

1. Open n8n
2. Click **Workflows** in the left sidebar
3. Click **Import from File**
4. Select the downloaded JSON file
5. Click **Import**

### Step 3: Configure Nodes

#### Webhook Node
- Default path: `multi-channel-form`
- Keep as POST method
- No authentication required for testing

#### Edit Fields Node
- No changes needed (handles data transformation)

#### Gmail Node (Send a message)
1. Click the node
2. Select credential: **Gmail account**
3. Update recipient email: Replace `s23500515@gmail.com` with your email
4. Customize subject and message if desired

#### Telegram Node (Send a text message)
1. Click the node
2. Select credential: **Telegram account**
3. Update Chat ID: Replace `5094673062` with your Chat ID
4. Customize message format if desired

#### Google Sheets Node (Append or update row)
1. Click the node
2. Select credential: **Google Sheets account**
3. Click **Select Document** dropdown
4. Choose your spreadsheet
5. Select **Sheet1**
6. Verify column mappings match your headers

### Step 4: Save and Activate

1. Click **Save** (top right)
2. Toggle **Active** switch to enable workflow
3. Copy the webhook URL from the Webhook node

## Testing

### Test 1: Manual Execution

1. Open the workflow
2. Click **Execute Workflow** button
3. Click on **Webhook** node
4. Click **Listen for Test Event**
5. Send a test request (see below)

### Test 2: cURL Request

```bash
curl -X POST https://your-n8n-instance.com/webhook/multi-channel-form \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test User",
    "email": "test@example.com",
    "phone": "+1234567890",
    "company": "Test Company",
    "service": "Automation Testing"
  }'
```

### Test 3: Verify Deliveries

- [ ] Check your Gmail inbox for email notification
- [ ] Check Telegram app for bot message
- [ ] Open Google Sheet to verify new row

### Common Test Issues

| Issue | Solution |
|-------|----------|
| 404 Not Found | Verify workflow is Active |
| Gmail not received | Check spam folder, verify OAuth |
| Telegram not delivered | Verify Chat ID, check bot token |
| Sheet not updated | Verify OAuth permissions, check Sheet ID |

## Production Deployment

### 1. Domain Setup (Self-Hosted)

```bash
# Example with nginx reverse proxy
server {
    listen 443 ssl;
    server_name n8n.yourdomain.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://localhost:5678;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
    }
}
```

### 2. Environment Variables

Create `.env` file (for self-hosted):

```env
N8N_HOST=n8n.yourdomain.com
N8N_PORT=5678
N8N_PROTOCOL=https
WEBHOOK_URL=https://n8n.yourdomain.com/
```

### 3. Update Webhook URLs

1. Update all webhook paths with production domain
2. Update OAuth redirect URIs in Google Cloud Console
3. Test thoroughly before going live

### 4. Security Hardening

- [ ] Enable webhook authentication
- [ ] Set up rate limiting
- [ ] Implement input validation
- [ ] Use HTTPS only
- [ ] Rotate credentials regularly

### 5. Monitoring

Set up monitoring for:
- Workflow execution success rate
- Email delivery status
- Telegram bot uptime
- Google Sheets API quota

## Next Steps

- Integrate with your website form
- Customize notification templates
- Add additional channels (Slack, SMS)
- Set up error alerting
- Monitor performance metrics

## Support

If you encounter issues:

1. Check n8n logs: Settings → Log Streaming
2. Review node error messages
3. Consult [n8n documentation](https://docs.n8n.io/)
4. Open an issue on GitHub
5. Email: salahbenbrika2@gmail.com

---

**Setup Complete! 🎉**
