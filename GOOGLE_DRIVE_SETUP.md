# Google Drive Backup Setup Instructions

This app uses a Cloudflare Worker for OAuth authentication, providing reliable session management without popup issues.

## Architecture

```
App (tax.matmcfad.com) ←→ Auth Worker (auth.matmcfad.com) ←→ Google OAuth
                                    ↓
                              KV Storage (refresh tokens)
```

## Prerequisites

- A Google account
- A [Cloudflare account](https://cloudflare.com) (free tier is sufficient)
- Access to [Google Cloud Console](https://console.cloud.google.com)

---

## Part 1: Google Cloud Console Setup

### 1. Create a Google Cloud Project

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Click the project dropdown at the top and select "New Project"
3. Name it "Alyx Tax Manager"
4. Click "Create"

### 2. Enable the Google Drive API

1. In your project, go to **APIs & Services** > **Library**
2. Search for "Google Drive API"
3. Click on it and press **Enable**

### 3. Configure OAuth Consent Screen

1. Go to **APIs & Services** > **OAuth consent screen**
2. Select **External** user type
3. Fill in required fields:
   - **App name**: Alyx Tax Manager
   - **User support email**: your email
   - **Developer contact**: your email
4. Click **Save and Continue**
5. On Scopes, click **Add or Remove Scopes**
6. Find and select: `https://www.googleapis.com/auth/drive.appdata`
7. Click **Update**, then **Save and Continue**
8. On Test Users, add your Google account email
9. Click **Save and Continue**

### 4. Create OAuth Client ID

1. Go to **APIs & Services** > **Credentials**
2. Click **Create Credentials** > **OAuth client ID**
3. Select **Web application**
4. Name it "Alyx Tax Manager Web Client"
5. Under **Authorized redirect URIs**, add:
   - `https://auth.matmcfad.com/auth/callback`
6. Click **Create**
7. **Save both the Client ID and Client Secret** - you'll need these for the Worker

---

## Part 2: Cloudflare Worker Setup

### 1. Create Cloudflare Account

1. Go to [cloudflare.com](https://cloudflare.com) and create a free account
2. Navigate to **Workers & Pages**

### 2. Create KV Namespace

1. Go to **Workers & Pages** > **KV**
2. Click **Create a namespace**
3. Name it `alyx-tax-tokens`
4. **Copy the namespace ID** - you'll need this

### 3. Deploy the Worker

From your local machine:

```bash
cd workers/auth-worker

# Install dependencies
npm install

# Update wrangler.toml with your account ID and KV namespace ID
# (Get account ID from Cloudflare dashboard URL)

# Set secrets
npx wrangler secret put GOOGLE_CLIENT_ID
# Paste your Google Client ID when prompted

npx wrangler secret put GOOGLE_CLIENT_SECRET
# Paste your Google Client Secret when prompted

# Deploy
npx wrangler deploy
```

### 4. Set Up Custom Domain

1. In Cloudflare Workers dashboard, click on your deployed worker
2. Go to **Settings** > **Triggers**
3. Click **Add Custom Domain**
4. Enter: `auth.matmcfad.com`
5. If your DNS is not on Cloudflare, add this CNAME record:
   - **Name**: `auth`
   - **Target**: `alyx-tax-auth.YOUR_SUBDOMAIN.workers.dev`

---

## Part 3: GitHub Actions (Auto-Deploy)

### 1. Create Cloudflare API Token

1. Go to [cloudflare.com/profile/api-tokens](https://dash.cloudflare.com/profile/api-tokens)
2. Click **Create Token**
3. Use the "Edit Cloudflare Workers" template
4. Click **Continue to Summary** > **Create Token**
5. Copy the token

### 2. Add GitHub Secrets

In your GitHub repository:

1. Go to **Settings** > **Secrets and variables** > **Actions**
2. Add these secrets:
   - `CLOUDFLARE_API_TOKEN`: Your API token from step 1
   - `CLOUDFLARE_ACCOUNT_ID`: Found in your Cloudflare dashboard URL

Now the Worker will auto-deploy when you push changes to `workers/auth-worker/`.

---

## How It Works

1. User clicks "Connect Google Drive" in the app
2. App redirects to Worker (`/auth/login`)
3. Worker redirects to Google consent screen
4. User grants permission
5. Google redirects back to Worker (`/auth/callback`)
6. Worker exchanges code for tokens, stores refresh token in KV
7. Worker sets session cookie and redirects back to app
8. App detects `?auth=success` and fetches access token from Worker
9. On future visits, app calls Worker to refresh token automatically

**Benefits:**
- No popups that browsers can block
- Stays logged in for 30 days (refresh token)
- Secure - client secret never exposed to browser
- Works across devices (different sessions)

---

## Troubleshooting

**"Session expired" on every refresh**
- Check that cookies are enabled
- Verify the Worker's custom domain is set up correctly
- Check browser console for CORS errors

**"Failed to get token after auth"**
- Verify the Worker secrets are set correctly
- Check Worker logs: `npx wrangler tail` in the worker directory

**"redirect_uri_mismatch" from Google**
- Make sure `https://auth.matmcfad.com/auth/callback` is in your OAuth client's Authorized redirect URIs
- Wait a few minutes after adding it (Google caches settings)

**Worker not deploying**
- Check GitHub Actions logs for errors
- Verify your Cloudflare API token has the right permissions

---

## Local Development

For local testing, you can run the Worker locally:

```bash
cd workers/auth-worker
npm run dev
```

This starts a local server at `http://localhost:8787`. Update the app's `AUTH_WORKER_URL` temporarily to test locally.

---

## Security Notes

- Refresh tokens are stored encrypted in Cloudflare KV
- Session cookies are HttpOnly, Secure, and SameSite=None
- The Client Secret is never exposed to the browser
- CORS is restricted to `https://tax.matmcfad.com`
- KV entries auto-expire after 30 days
