# 🔧 Complete Setup Guide - Supabase Self-Hosted with Mobile Deep Linking

This guide will walk you through setting up a complete, production-ready Supabase instance with custom email templates and mobile deep linking.

## 📋 Prerequisites

Before starting, ensure you have:

- **Docker & Docker Compose** installed
- **Domain name** for email sending
- **SMTP email provider** (Gmail, SendGrid, etc.)
- **Mobile app** with URL scheme configured
- **Coolify instance** (optional, for managed deployment)

## 🚀 Step 1: Initial Setup

### Clone and Prepare

```bash
# Clone the repository
git clone https://github.com/ykarateke/Supabase-Self-Hosted-with-Mobile-Deep-Linking.git
cd Supabase-Self-Hosted-with-Mobile-Deep-Linking

# Repository includes:
# - docker-compose-coolify.yml
# - .env.example
# - mail-template/ folder with custom HTML email templates
# - Complete documentation
```

### Environment Configuration

```bash
# Copy environment template
cp .env.example .env

# Edit with your settings
nano .env
```

## 📧 Step 2: SMTP Configuration

Choose your email provider and configure SMTP settings:

### Option A: Gmail

1. Enable 2-factor authentication on your Google account
2. Generate an app-specific password
3. Configure in `.env`:

```env
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password
SMTP_ADMIN_EMAIL=your-email@gmail.com
SMTP_SENDER_NAME=Your App Name
```

### Option B: SendGrid

1. Create SendGrid account
2. Generate API key
3. Configure in `.env`:

```env
SMTP_HOST=smtp.sendgrid.net
SMTP_PORT=587
SMTP_USER=apikey
SMTP_PASS=SG.your-sendgrid-api-key-here
SMTP_ADMIN_EMAIL=noreply@yourdomain.com
SMTP_SENDER_NAME=Your App Name
```

### Option C: Custom Provider

```env
SMTP_HOST=mail.yourdomain.com
SMTP_PORT=587
SMTP_USER=noreply@yourdomain.com
SMTP_PASS=your-smtp-password
SMTP_ADMIN_EMAIL=noreply@yourdomain.com
SMTP_SENDER_NAME=Your Company Name
```

### Test SMTP Configuration

```bash
# Test SMTP connection
telnet smtp.gmail.com 587
# Should connect successfully
```

## 📱 Step 3: Mobile Deep Link Configuration

### Configure Your App Scheme

Replace `yourapp` with your actual app's URL scheme in `.env`:

```env
# Replace 'yourapp' with your actual app scheme (e.g., 'myawesomeapp')
ADDITIONAL_REDIRECT_URLS=myawesomeapp:///reset-password-landing,myawesomeapp:///auth-callback
GOTRUE_SITE_URL=myawesomeapp:///
GOTRUE_URI_ALLOW_LIST=myawesomeapp:///**

# Deep link paths
MAILER_URLPATHS_CONFIRMATION=myawesomeapp:///auth-callback
MAILER_URLPATHS_EMAIL_CHANGE=myawesomeapp:///auth-callback
MAILER_URLPATHS_INVITE=myawesomeapp:///auth-callback
MAILER_URLPATHS_RECOVERY=myawesomeapp:///reset-password-landing
```

### Android Configuration

Add intent filters to `android/app/src/main/AndroidManifest.xml`:

```xml
<activity
    android:name=\".MainActivity\"
    android:exported=\"true\"
    android:launchMode=\"singleTop\"
    android:theme=\"@style/LaunchTheme\">

    <!-- Existing intent filter for app launch -->
    <intent-filter>
        <action android:name=\"android.intent.action.MAIN\"/>
        <category android:name=\"android.intent.category.LAUNCHER\"/>
    </intent-filter>

    <!-- Deep link for auth callback -->
    <intent-filter>
        <action android:name=\"android.intent.action.VIEW\" />
        <category android:name=\"android.intent.category.DEFAULT\" />
        <category android:name=\"android.intent.category.BROWSABLE\" />
        <data android:scheme=\"myawesomeapp\" android:host=\"\" android:pathPrefix=\"/auth-callback\"/>
    </intent-filter>

    <!-- Deep link for password reset -->
    <intent-filter>
        <action android:name=\"android.intent.action.VIEW\" />
        <category android:name=\"android.intent.category.DEFAULT\" />
        <category android:name=\"android.intent.category.BROWSABLE\" />
        <data android:scheme=\"myawesomeapp\" android:host=\"\" android:pathPrefix=\"/reset-password-landing\"/>
    </intent-filter>
</activity>
```

### iOS Configuration

Add URL schemes to `ios/Runner/Info.plist`:

```xml
<dict>
    <!-- Existing configuration -->

    <key>CFBundleURLTypes</key>
    <array>
        <dict>
            <key>CFBundleURLName</key>
            <string>myawesomeapp</string>
            <key>CFBundleURLSchemes</key>
            <array>
                <string>myawesomeapp</string>
            </array>
        </dict>
    </array>
</dict>
```

### Flutter Deep Link Handler

```dart
import 'package:app_links/app_links.dart';
import 'package:supabase_flutter/supabase_flutter.dart';

class DeepLinkHandler {
  static final _appLinks = AppLinks();

  static void initialize() {
    _appLinks.uriLinkStream.listen((uri) {
      handleDeepLink(uri.toString());
    });
  }

  static void handleDeepLink(String link) {
    final uri = Uri.parse(link);
    final token = uri.queryParameters['token'];
    final type = uri.queryParameters['type'];

    switch (type) {
      case 'recovery':
        // Navigate to password reset screen
        navigateToPasswordReset(token);
        break;
      case 'signup':
      case 'email_change':
      case 'invite':
        // Handle other auth types
        handleAuthCallback(token, type);
        break;
    }
  }

  static void navigateToPasswordReset(String? token) {
    if (token != null) {
      // Navigate to your password reset screen
      // Pass the token for verification
    }
  }

  static void handleAuthCallback(String? token, String? type) {
    if (token != null) {
      // Handle the authentication token
      Supabase.instance.client.auth.verifyOTP(
        token: token,
        type: OtpType.values.firstWhere((e) => e.name == type),
      );
    }
  }
}
```

## 🎨 Step 4: Customize Email Templates

### Email Subjects

Update email subjects in `.env`:

```env
MAILER_SUBJECTS_CONFIRMATION=MyApp: Verify Your Account
MAILER_SUBJECTS_EMAIL_CHANGE=MyApp: Confirm Email Change
MAILER_SUBJECTS_INVITE=MyApp: You've Been Invited
MAILER_SUBJECTS_MAGIC_LINK=MyApp: Your Login Link
MAILER_SUBJECTS_RECOVERY=MyApp: Reset Your Password
```

### Branding

```env
STUDIO_DEFAULT_ORGANIZATION=My Company Name
STUDIO_DEFAULT_PROJECT=My Project Name
```

### Template Customization

1. Navigate to `mail-template/` directory
2. Edit HTML files with your branding:
   - Update logo URLs
   - Change color schemes
   - Modify text content
   - Add your company information

Example customization in `recovery.html`:
```html
<!-- Replace the logo URL -->
<img alt=\"Your App Logo\" src=\"https://yourdomain.com/logo.png\" />

<!-- Update company name in footer -->
<div>© 2024 Your Company Name. All rights reserved.</div>

<!-- Change primary color -->
<style>
  .primary-color { background-color: #your-brand-color; }
</style>
```

## 🐳 Step 5: Deploy with Docker Compose

### Standard Deployment

```bash
# Start all services
docker-compose -f docker-compose-coolify.yml up -d

# Check service status
docker-compose ps

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

### Production Deployment

```bash
# Pull latest images
docker-compose pull

# Start in production mode
docker-compose -f docker-compose-coolify.yml up -d --remove-orphans

# Enable auto-restart
docker-compose -f docker-compose-coolify.yml up -d --restart unless-stopped
```

## 🌐 Step 6: Coolify Deployment (Optional)

### Coolify Setup

1. **Create New Project** in Coolify
2. **Add Service** → Docker Compose
3. **Upload Files**:
   - `docker-compose-coolify.yml`
   - `.env` (with your configuration)
   - `mail-template/` folder

### Coolify Configuration

1. **Environment Variables**: Import from `.env` file
2. **Domain Settings**: Configure your domain
3. **SSL Certificate**: Enable automatic SSL
4. **Deployment**: Deploy the stack

### Coolify File Structure

```
/data/coolify/services/your-service-id/
├── docker-compose-coolify.yml
├── .env
└── volumes/
    └── email-templates/
        ├── confirmation.html
        ├── recovery.html
        ├── invite.html
        ├── magic_link.html
        ├── email_change.html
        └── reauthentication.html
```

## 🧪 Step 7: Testing & Verification

### Test Email Templates

1. **Access Supabase Studio**:
   - URL: `http://localhost:3000` (or your domain)
   - Login with admin credentials

2. **Test Password Reset**:
   - Go to Authentication → Users
   - Create a test user or select existing
   - Click \"Send recovery email\"
   - Check email for proper formatting and deep link

3. **Verify Deep Link Format**:
   ```
   Expected: myawesomeapp:///reset-password-landing?token=...&type=recovery
   Not:      https://your-domain.com/auth/v1/verify?token=...&redirect_to=...
   ```

### Test Mobile Deep Links

1. **Send Reset from App**:
   ```dart
   await Supabase.instance.client.auth.resetPasswordForEmail(
     'test@example.com'
   );
   ```

2. **Check Email**: Verify deep link format
3. **Test Link**: Tap the email button/link
4. **Verify App Opens**: Confirm your app opens with correct token

### Test SMTP Connection

```bash
# Test SMTP manually
telnet your-smtp-host 587

# Expected output:
# Trying xxx.xxx.xxx.xxx...
# Connected to smtp.example.com.
# Escape character is '^]'.
# 220 smtp.example.com ESMTP
```

### Test Email Template Service

```bash
# Test template availability
curl http://localhost:8080/recovery.html

# Should return HTML content
```

## 🔒 Step 8: Security Configuration

### SSL/TLS Setup

For production, ensure SSL is configured:

```yaml
# In docker-compose-coolify.yml (if needed)
environment:
  - KONG_SSL_CERT=/path/to/cert.pem
  - KONG_SSL_CERT_KEY=/path/to/key.pem
```

### Firewall Configuration

```bash
# Allow only necessary ports
ufw allow 22/tcp    # SSH
ufw allow 80/tcp    # HTTP
ufw allow 443/tcp   # HTTPS
ufw deny 5432/tcp   # PostgreSQL (internal only)
ufw deny 3000/tcp   # Studio (use reverse proxy)
```

### Backup Configuration

```bash
# Database backup script
#!/bin/bash
docker exec supabase-db pg_dump -U postgres postgres > backup_$(date +%Y%m%d_%H%M%S).sql

# Schedule with cron
crontab -e
# Add: 0 2 * * * /path/to/backup-script.sh
```

## 📊 Step 9: Monitoring & Logs

### Log Management

```bash
# View all logs
docker-compose logs

# View specific service logs
docker-compose logs supabase-auth
docker-compose logs email-templates

# Follow logs in real-time
docker-compose logs -f supabase-auth

# Limit log output
docker-compose logs --tail=100 supabase-auth
```

### Health Checks

```bash
# Check service health
docker-compose ps

# Test API endpoints
curl http://localhost:8000/health
curl http://localhost:3000/api/health
```

### Performance Monitoring

Monitor key metrics:
- Database connections: `SELECT count(*) FROM pg_stat_activity;`
- Email delivery rates
- Authentication success rates
- Deep link conversion rates

## 🔧 Troubleshooting

### Common Issues

#### Email Not Sending

**Problem**: Users not receiving emails

**Solutions**:
1. Check SMTP credentials in `.env`
2. Verify SMTP port and security settings
3. Check auth service logs: `docker-compose logs supabase-auth`
4. Test SMTP connection manually
5. Check email provider limits/quotas

#### Deep Links Not Working

**Problem**: App doesn't open from email links

**Solutions**:
1. Verify URL scheme in mobile app configuration
2. Check `GOTRUE_URI_ALLOW_LIST` in `.env`
3. Test deep link manually: `adb shell am start -W -a android.intent.action.VIEW -d \"myawesomeapp:///test\"`
4. Verify intent filters (Android) or URL schemes (iOS)

#### Database Connection Issues

**Problem**: Cannot connect to database

**Solutions**:
1. Check database logs: `docker-compose logs supabase-db`
2. Verify database credentials
3. Check network connectivity between services
4. Restart database service: `docker-compose restart supabase-db`

#### Email Templates Not Loading

**Problem**: Emails show broken formatting

**Solutions**:
1. Check email-templates service: `docker-compose logs email-templates`
2. Verify template files exist in `mail-template/` directory
3. Test template endpoint: `curl http://localhost:8080/recovery.html`
4. Restart email-templates service: `docker-compose restart email-templates`

### Getting Help

1. **Check Logs**: Always start with `docker-compose logs`
2. **Review Configuration**: Double-check `.env` settings
3. **Test Components**: Test SMTP, deep links, and templates individually
4. **Community Support**:
   - [Project GitHub Issues](https://github.com/ykarateke/Supabase-Self-Hosted-with-Mobile-Deep-Linking/issues)
   - [Supabase GitHub Issues](https://github.com/supabase/supabase/issues)
   - [Docker Compose Documentation](https://docs.docker.com/compose/)

## ✅ Production Checklist

Before going live:

- [ ] SSL/TLS certificates configured
- [ ] Strong passwords for all services
- [ ] `.env` file secured and not in version control
- [ ] Firewall rules configured
- [ ] Database backups automated
- [ ] Monitoring and alerting set up
- [ ] Email deliverability tested
- [ ] Deep links tested on real devices
- [ ] Performance testing completed
- [ ] Security scan performed
- [ ] Documentation updated for your team

## 🎯 Next Steps

After successful deployment:

1. **Monitor Performance**: Set up monitoring dashboards
2. **Scale as Needed**: Adjust resource limits based on usage
3. **Regular Backups**: Implement automated backup strategy
4. **Security Updates**: Keep Docker images updated
5. **User Training**: Train your team on the admin interface
6. **Analytics**: Set up user analytics and email tracking

---

**🎉 Congratulations!** You now have a fully functional, self-hosted Supabase instance with custom email templates and mobile deep linking. Your users will enjoy seamless authentication flows from email to your mobile app.