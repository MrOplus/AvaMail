# AvaMail

A self-hosted email management system built on Cloudflare Workers. Receive emails on your custom domain, store them in a database, and send emails through Brevo - all running on Cloudflare's edge network.

## Features

- **Receive Emails**: Use Cloudflare Email Routing to receive emails on your custom domain
- **Send Emails**: Send emails through Brevo's reliable SMTP API
- **Attachment Support**: Send and receive emails with attachments (up to 5MB per file)
- **Modern UI**: Vue 3 SPA with a clean, responsive interface
- **Secure**: PBKDF2 password hashing, rate limiting, security headers
- **Serverless**: Runs entirely on Cloudflare Workers with D1 database
- **Optional Forwarding**: Optionally forward received emails to your personal inbox

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Cloudflare Edge                          │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   Email     │    │   Worker    │    │     D1      │     │
│  │   Routing   │───▶│   (Hono)    │◀──▶│  Database   │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
│         │                  │                               │
│         │                  │                               │
│         ▼                  ▼                               │
│  ┌─────────────┐    ┌─────────────┐                       │
│  │  Incoming   │    │   Vue SPA   │                       │
│  │   Emails    │    │  (Frontend) │                       │
│  └─────────────┘    └─────────────┘                       │
└─────────────────────────────────────────────────────────────┘
                             │
                             ▼
                    ┌─────────────┐
                    │    Brevo    │
                    │  (Sending)  │
                    └─────────────┘
```

## Prerequisites

Before you begin, make sure you have:

1. **Cloudflare Account** with a domain added
2. **Brevo Account** (free tier available at [brevo.com](https://www.brevo.com))
3. **Node.js 18+** and **pnpm** installed
4. **Wrangler CLI** installed (`npm install -g wrangler`)

## Quick Start

### 1. Clone and Install

```bash
git clone https://github.com/MrOplus/AvaMail.git
cd AvaMail
pnpm install
```

### 2. Configure Cloudflare

Login to Wrangler:

```bash
wrangler login
```

Create a D1 database:

```bash
cd packages/worker
wrangler d1 create avamail-db
```

Copy the example config and update with your database ID:

```bash
cp wrangler.toml.example wrangler.toml
```

Then edit `wrangler.toml` and replace `YOUR_DATABASE_ID_HERE` with the ID from the previous command:

```toml
[[d1_databases]]
binding = "DB"
database_name = "avamail-db"
database_id = "your-actual-database-id"
```

### 3. Initialize Database

Run the database migrations:

```bash
wrangler d1 execute avamail-db --remote --file=src/db/schema.sql
```

### 4. Build Frontend

```bash
cd ../../frontend
pnpm run build
```

### 5. Deploy

```bash
cd ../packages/worker
pnpm run deploy
```

Your application will be deployed to `https://avamail.<your-subdomain>.workers.dev`

## Wrangler Configuration

The `wrangler.toml` file is gitignored to prevent accidentally committing your database ID. Copy the example file to get started:

```bash
cd packages/worker
cp wrangler.toml.example wrangler.toml
```

The configuration file contains:

```toml
name = "avamail"                    # Worker name (can be customized)
main = "src/index.ts"
compatibility_date = "2024-01-01"

# D1 Database - Replace with your actual database ID
[[d1_databases]]
binding = "DB"
database_name = "avamail-db"
database_id = "YOUR_DATABASE_ID_HERE"  # From: wrangler d1 create avamail-db

# Static assets from Vue build
[assets]
directory = "../../frontend/dist"

# Environment variables
[vars]
APP_NAME = "AvaMail"
```

### Required Configuration

| Setting | Description | How to Get |
|---------|-------------|------------|
| `database_id` | Your D1 database UUID | Run `wrangler d1 create avamail-db` and copy the ID |

### Optional Customization

| Setting | Default | Description |
|---------|---------|-------------|
| `name` | `avamail` | Worker name (affects the URL: `name.subdomain.workers.dev`) |
| `APP_NAME` | `AvaMail` | Application display name |

> **Important**: Never commit sensitive data to `wrangler.toml`. API keys and secrets are stored securely in the D1 database after initial setup through the UI.

## Configuration Steps

### Step 1: Initial Setup

1. Open your deployed application URL
2. Create your admin password (minimum 8 characters)
3. You'll be logged in automatically

### Step 2: Configure Cloudflare Email Routing

1. Go to **Settings** → **Cloudflare** tab
2. Create a Cloudflare API Token with the required permissions:

   **Creating an API Token:**
   1. Go to [Cloudflare Dashboard](https://dash.cloudflare.com/profile/api-tokens) → **API Tokens**
   2. Click **Create Token**
   3. Select **Create Custom Token**
   4. Add the following permissions:

   **Account Permissions:**
   | Permission | Access Level |
   |------------|--------------|
   | Email Routing Addresses | Edit |

   **Zone Permissions:**
   | Permission | Access Level |
   |------------|--------------|
   | Zone | Read |
   | DNS | Edit |
   | Email Routing Rules | Edit |
   | Zone Settings | Edit |

   5. Set **Zone Resources** to: `Include` → `All zones` (or select specific zone)
   6. Set **Account Resources** to: `Include` → your account
   7. Click **Continue to summary** → **Create Token**
   8. Copy the token (you won't see it again!)

3. Enter your Cloudflare credentials in AvaMail:

   | Field | Description | Where to Find |
   |-------|-------------|---------------|
   | API Token | Custom API token | Created above with required scopes |
   | Account ID | Your account identifier | Dashboard sidebar or any zone URL |
   | Domain | Your email domain | e.g., `example.com` |
   | Forwarding Email | (Optional) Backup email | Your personal email for backup forwarding |

4. Click **Save Cloudflare Settings**
5. Click **Fix Email Routing** to ensure emails route to the worker

### Step 3: Configure Brevo (Sending)

1. Go to **Settings** → **Brevo** tab
2. Get your Brevo API key:
   - Log into [Brevo](https://app.brevo.com)
   - Go to **SMTP & API** → **API Keys**
   - Create a new API key or use existing one
3. Enter your API key and save

### Step 4: Verify Domain in Brevo

To send emails from your domain:

1. Go to [Brevo Senders & Domains](https://app.brevo.com/senders/domain/list)
2. Add your domain
3. Add the required DNS records (SPF, DKIM, DMARC)
4. Verify the domain

### Step 5: Add Email Addresses

1. Go to **Settings** → **Addresses** tab
2. Add email addresses you want to use (e.g., `contact@yourdomain.com`)
3. Set a default sending address

### Step 6: Complete Setup

1. Once Cloudflare and Brevo are configured, click **Complete Setup**
2. You're ready to send and receive emails!

## DNS Records

For email to work properly, ensure these DNS records are set:

### Cloudflare Email Routing (MX Records)

Cloudflare will automatically configure these, but verify:

| Type | Name | Value | Priority |
|------|------|-------|----------|
| MX | @ | `route1.mx.cloudflare.net` | 69 |
| MX | @ | `route2.mx.cloudflare.net` | 15 |
| MX | @ | `route3.mx.cloudflare.net` | 21 |
| TXT | @ | `v=spf1 include:_spf.mx.cloudflare.net ~all` | - |

### Brevo (for sending)

Add the records provided by Brevo for your domain:

| Type | Name | Purpose |
|------|------|---------|
| TXT | @ | SPF record |
| TXT | mail._domainkey | DKIM record |
| TXT | _dmarc | DMARC record |

## Usage

### Receiving Emails

Emails sent to any address at your domain (e.g., `anything@yourdomain.com`) will be:
1. Received by Cloudflare Email Routing
2. Processed by the worker
3. Stored in the D1 database
4. (Optional) Forwarded to your personal email

### Sending Emails

1. Click **Compose** in the sidebar
2. Select your "From" address
3. Enter recipient, subject, and message
4. Add attachments if needed (max 5MB per file, 10MB total)
5. Click **Send**

### Managing Emails

- **Inbox**: View received emails
- **Sent**: View sent emails
- **Star**: Mark important emails
- **Delete**: Remove emails

## Development

### Local Development

```bash
# Start frontend dev server
cd frontend
pnpm run dev

# In another terminal, start worker dev server
cd packages/worker
pnpm run dev
```

### Project Structure

```
AvaMail/
├── frontend/                 # Vue 3 SPA
│   ├── src/
│   │   ├── api/             # API client
│   │   ├── stores/          # Pinia stores
│   │   ├── views/           # Page components
│   │   └── router/          # Vue Router
│   └── dist/                # Built assets
│
├── packages/
│   ├── worker/              # Cloudflare Worker
│   │   ├── src/
│   │   │   ├── api/         # API routes
│   │   │   ├── db/          # Database queries
│   │   │   ├── lib/         # Utilities
│   │   │   └── email-handler.ts
│   │   └── wrangler.toml
│   │
│   ├── cloudflare-email-api/ # Cloudflare API wrapper
│   └── mailgun-api/          # Brevo API wrapper
│
└── README.md
```

## Security

### Features

- **PBKDF2 Password Hashing**: 100,000 iterations with SHA-256
- **Rate Limiting**: Login attempts limited to 5 per 15 minutes
- **Security Headers**: X-Frame-Options, X-Content-Type-Options, etc.
- **Session Management**: Secure token-based authentication
- **Input Validation**: All inputs validated and sanitized
- **Timing-Safe Comparisons**: Prevents timing attacks

### Recommendations

For production use:

1. **Custom Domain**: Use your own domain instead of `workers.dev`
2. **HTTPS Only**: Cloudflare Workers are HTTPS by default
3. **Strong Password**: Use a strong, unique admin password
4. **Regular Updates**: Keep dependencies updated
5. **Backup**: Regularly export your D1 database

## Troubleshooting

### Emails Not Being Received

1. Verify DNS records are correct
2. Check Cloudflare Email Routing is enabled
3. Click **Fix Email Routing** in Settings → Cloudflare
4. Check the worker logs: `wrangler tail`

### Cannot Send Emails

1. Verify Brevo API key is correct
2. Ensure your domain is verified in Brevo
3. Check sender address is added to Brevo senders

### Login Issues

1. Clear browser cache and cookies
2. If locked out, wait 15 minutes for rate limit reset
3. For password reset, manually update D1 database

### Rate Limit Errors

- Login: 5 attempts per 15 minutes
- Setup: 3 attempts per hour
- Wait for the specified time before retrying

## API Reference

### Authentication

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/auth/status` | GET | Check auth status |
| `/api/auth/login` | POST | Login with password |
| `/api/auth/logout` | POST | Logout |
| `/api/auth/setup` | POST | Initial password setup |
| `/api/auth/change-password` | POST | Change password |

### Emails

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/emails` | GET | List emails |
| `/api/emails/stats` | GET | Get email statistics |
| `/api/emails/:id` | GET | Get single email |
| `/api/emails/send` | POST | Send email |
| `/api/emails/:id/attachments/:attachmentId` | GET | Download attachment |

### Setup

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/setup/status` | GET | Get setup status |
| `/api/setup/cloudflare` | POST | Configure Cloudflare |
| `/api/setup/brevo` | POST | Configure Brevo |
| `/api/setup/cloudflare/worker-routing` | POST | Fix email routing |

## License

MIT License - See LICENSE file for details.

## Support

For issues and feature requests, please visit [GitHub Issues](https://github.com/MrOplus/AvaMail/issues).
