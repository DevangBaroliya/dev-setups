# Mailpit Setup

Mailpit is used for local email testing. It works like a fake SMTP server and captures emails in a local web inbox instead of sending them to real users.

## Local Development Setup

### 1. Create `docker-compose.yml`

```yaml
services:
  mailpit:
    image: axllent/mailpit:latest
    container_name: mailpit
    restart: unless-stopped
    ports:
      - "8025:8025" # Web UI
      - "1025:1025" # SMTP
    volumes:
      - ./mailpit-data:/data
    environment:
      MP_DATABASE: /data/mailpit.db
      MP_UI_AUTH_FILE: /data/ui-auth
      TZ: Asia/Kolkata
```

### 2. Create UI auth file

```bash
mkdir -p mailpit-data
echo "admin:admin123" > mailpit-data/ui-auth
```

### 3. Start Mailpit

```bash
docker compose up -d
```

### 4. Open Mailpit UI

```txt
http://localhost:8025
```

Login:

```txt
Username: admin
Password: admin123
```

## Node.js / TypeScript SMTP Config

Use this in your local `.env`:

```env
SMTP_HOST=localhost
SMTP_PORT=1025
SMTP_USERNAME=
SMTP_PASSWORD=
MAIL_FROM="My App <noreply@example.com>"
```

For local development, SMTP username/password is not required.

Example Nodemailer config:

```ts
import nodemailer from "nodemailer";

export const transporter = nodemailer.createTransport({
  host: process.env.SMTP_HOST,
  port: Number(process.env.SMTP_PORT),
  secure: false,
});
```

## Optional: Enable SMTP Username and Password

Use this only if you want your app to authenticate with Mailpit SMTP.

Update `docker-compose.yml`:

```yaml
services:
  mailpit:
    image: axllent/mailpit:latest
    container_name: mailpit
    restart: unless-stopped
    ports:
      - "8025:8025"
      - "1025:1025"
    volumes:
      - ./mailpit-data:/data
    environment:
      MP_DATABASE: /data/mailpit.db
      MP_UI_AUTH_FILE: /data/ui-auth
      MP_SMTP_AUTH_FILE: /data/smtp-auth
      MP_SMTP_AUTH_ALLOW_INSECURE: "true"
      TZ: Asia/Kolkata
```

Create SMTP auth file:

```bash
echo "devuser:devpass" > mailpit-data/smtp-auth
```

Update app `.env`:

```env
SMTP_HOST=localhost
SMTP_PORT=1025
SMTP_USERNAME=devuser
SMTP_PASSWORD=devpass
MAIL_FROM="My App <noreply@example.com>"
```

Nodemailer config:

```ts
import nodemailer from "nodemailer";

export const transporter = nodemailer.createTransport({
  host: process.env.SMTP_HOST,
  port: Number(process.env.SMTP_PORT),
  secure: false,
  auth: {
    user: process.env.SMTP_USERNAME,
    pass: process.env.SMTP_PASSWORD,
  },
});
```

Note: `MP_SMTP_AUTH_ALLOW_INSECURE=true` is for local development only because SMTP auth is used without TLS.

## Git Setup

Do not push local Mailpit data or secrets.

Add this to `.gitignore`:

```gitignore
.env
mailpit-data/
```

You can push:

```txt
docker-compose.yml
.env.example
```

Do not push:

```txt
.env
mailpit-data/
```

Example `.env.example`:

```env
SMTP_HOST=localhost
SMTP_PORT=1025
SMTP_USERNAME=
SMTP_PASSWORD=
MAIL_FROM="My App <noreply@example.com>"
```

## Production Setup

Mailpit should not be used as the real production email sender.

For production, use a transactional email provider such as:

```txt
AWS SES
SendGrid
Postmark
Mailgun
Mailtrap Email Sending
```

Production flow:

```txt
Node.js App
  -> Nodemailer
  -> AWS SES / SendGrid / Postmark
  -> Real user inbox
```

Production `.env` example using AWS SES SMTP:

```env
SMTP_HOST=email-smtp.ap-south-1.amazonaws.com
SMTP_PORT=587
SMTP_USERNAME=your_ses_smtp_username
SMTP_PASSWORD=your_ses_smtp_password
MAIL_FROM="My App <noreply@yourdomain.com>"
```

Production Nodemailer config:

```ts
import nodemailer from "nodemailer";

export const transporter = nodemailer.createTransport({
  host: process.env.SMTP_HOST,
  port: Number(process.env.SMTP_PORT),
  secure: false,
  auth: {
    user: process.env.SMTP_USERNAME,
    pass: process.env.SMTP_PASSWORD,
  },
});
```

Before going live, make sure to:

```txt
Verify your domain
Configure SPF
Configure DKIM
Configure DMARC
Use a real sender email like noreply@yourdomain.com
Keep SMTP credentials in environment variables or a secret manager
```

## Useful Commands

Start Mailpit:

```bash
docker compose up -d
```

Stop Mailpit:

```bash
docker compose down
```

View logs:

```bash
docker compose logs -f mailpit
```

Restart Mailpit:

```bash
docker compose restart mailpit
```

Remove local Mailpit data:

```bash
rm -rf mailpit-data
```
