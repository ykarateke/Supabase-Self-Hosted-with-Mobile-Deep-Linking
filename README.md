# 🚀 Supabase Self-Hosted with Mobile Deep Linking

[![GitHub stars](https://img.shields.io/github/stars/ykarateke/Supabase-Self-Hosted-with-Mobile-Deep-Linking?style=flat-square)](https://github.com/ykarateke/Supabase-Self-Hosted-with-Mobile-Deep-Linking/stargazers)
[![GitHub issues](https://img.shields.io/github/issues/ykarateke/Supabase-Self-Hosted-with-Mobile-Deep-Linking?style=flat-square)](https://github.com/ykarateke/Supabase-Self-Hosted-with-Mobile-Deep-Linking/issues)
[![GitHub license](https://img.shields.io/github/license/ykarateke/Supabase-Self-Hosted-with-Mobile-Deep-Linking?style=flat-square)](https://github.com/ykarateke/Supabase-Self-Hosted-with-Mobile-Deep-Linking/blob/main/LICENSE)

A complete, production-ready Supabase self-hosted setup with custom email templates and mobile deep linking support. Perfect for mobile apps that need secure authentication with seamless user experience.

## ✨ Features

- 🔐 **Self-hosted Supabase** - Complete control over your data
- 📧 **Custom Email Templates** - Beautiful, branded authentication emails
- 📱 **Mobile Deep Linking** - Seamless mobile app integration
- 🔒 **Secure Configuration** - Best practices for production deployment
- 🐳 **Docker Compose** - Easy deployment with Coolify support
- 🎨 **Customizable UI** - Branded email templates and subjects

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Mobile App    │    │   Web Client    │    │   Dashboard     │
│                 │    │                 │    │                 │
│ yourapp://auth  │    │ HTTP Requests   │    │ Admin Panel     │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          └──────────────────────┼──────────────────────┘
                                 │
                    ┌─────────────▼─────────────┐
                    │       Kong Gateway        │
                    │     (API Router)          │
                    └─────────────┬─────────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                       │                        │
┌───────▼───────┐    ┌──────────▼──────────┐    ┌────────▼────────┐
│ Supabase Auth │    │  Email Templates    │    │   PostgreSQL    │
│   (GoTrue)    │    │     (Nginx)         │    │   Database      │
│               │    │                     │    │                 │
│ JWT, Sessions │    │ HTML Templates      │    │ User Data       │
└───────────────┘    └─────────────────────┘    └─────────────────┘
```

## 🚀 Quick Start

### Prerequisites

- Docker & Docker Compose
- Domain name (for email sending)
- SMTP email provider
- Mobile app with URL scheme configured

### 1. Clone & Configure

```bash
# Clone the repository
git clone https://github.com/ykarateke/Supabase-Self-Hosted-with-Mobile-Deep-Linking.git
cd Supabase-Self-Hosted-with-Mobile-Deep-Linking

# Copy environment template
cp .env.example .env

# Edit configuration
nano .env
```

### 2. Configure Email & Deep Linking

Update `.env` with your settings:

```env
# SMTP Email Configuration
SMTP_HOST=mail.yourdomain.com
SMTP_USER=noreply@yourdomain.com
SMTP_PASS=your_smtp_password

# Mobile Deep Linking (Replace 'yourapp' with your app scheme)
ADDITIONAL_REDIRECT_URLS=yourapp:///reset-password-landing,yourapp:///auth-callback
GOTRUE_SITE_URL=yourapp:///
GOTRUE_URI_ALLOW_LIST=yourapp:///**
```

### 3. Deploy

```bash
# Start all services
docker-compose -f docker-compose-coolify.yml up -d

# Check status
docker-compose ps
```

### 4. Access

- **Supabase Studio**: `http://localhost:3000`
- **API Gateway**: `http://localhost:8000`
- **Database**: `localhost:5432`

## 📧 Email Templates

This setup includes custom HTML email templates with mobile deep linking support:

### Available Templates

- `confirmation.html` - Account verification
- `recovery.html` - Password reset
- `invite.html` - User invitations
- `magic_link.html` - Magic link login
- `email_change.html` - Email change confirmation
- `reauthentication.html` - Re-authentication

### Template Features

- 🎨 **Branded Design** - Clean, professional appearance
- 📱 **Mobile Optimized** - Responsive email layout
- 🔗 **Deep Link Support** - Direct app opening from emails
- 🔒 **No Redirect Exposure** - Secure, no internal URLs exposed
- 🌐 **Multi-language Ready** - Easy to customize text

### How It Works

1. **User Action**: User requests password reset from mobile app
2. **Email Generation**: Supabase generates email with deep link
3. **Template Rendering**: Custom HTML template is used
4. **Deep Link**: Email contains `yourapp:///reset-password-landing?token=...`
5. **App Opening**: User taps link, mobile app opens directly
6. **Token Processing**: App handles authentication token

## 🔧 Configuration Guide

### SMTP Providers

#### Gmail
```env
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password
```

#### SendGrid
```env
SMTP_HOST=smtp.sendgrid.net
SMTP_PORT=587
SMTP_USER=apikey
SMTP_PASS=your-sendgrid-api-key
```

#### Custom Provider
```env
SMTP_HOST=mail.yourdomain.com
SMTP_PORT=587
SMTP_USER=noreply@yourdomain.com
SMTP_PASS=your-password
```

### Mobile App Deep Linking

#### Android Configuration

Add to `android/app/src/main/AndroidManifest.xml`:

```xml
<intent-filter>
    <action android:name=\"android.intent.action.VIEW\" />
    <category android:name=\"android.intent.category.DEFAULT\" />
    <category android:name=\"android.intent.category.BROWSABLE\" />
    <data android:scheme=\"yourapp\" android:host=\"\" android:pathPrefix=\"/auth-callback\"/>
</intent-filter>

<intent-filter>
    <action android:name=\"android.intent.action.VIEW\" />
    <category android:name=\"android.intent.category.DEFAULT\" />
    <category android:name=\"android.intent.category.BROWSABLE\" />
    <data android:scheme=\"yourapp\" android:host=\"\" android:pathPrefix=\"/reset-password-landing\"/>
</intent-filter>
```

#### iOS Configuration

Add to `ios/Runner/Info.plist`:

```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLName</key>
        <string>yourapp</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>yourapp</string>
        </array>
    </dict>
</array>
```

#### Flutter Integration

```dart
// Password reset request
await Supabase.instance.client.auth.resetPasswordForEmail(email);

// Handle deep link
void handleDeepLink(String link) {
  final uri = Uri.parse(link);
  final token = uri.queryParameters['token'];
  final type = uri.queryParameters['type'];

  if (type == 'recovery') {
    // Navigate to password reset screen
    Navigator.pushNamed(context, '/reset-password', arguments: token);
  }
}
```

## 🛠️ Customization

### Email Templates

1. Navigate to `mail-template/` directory
2. Edit HTML files with your branding
3. Update color scheme, logos, and text
4. Restart email-templates service: `docker-compose restart email-templates`

### Email Subjects

Update in `.env`:

```env
MAILER_SUBJECTS_CONFIRMATION=YourApp: Verify Your Account
MAILER_SUBJECTS_RECOVERY=YourApp: Reset Your Password
MAILER_SUBJECTS_INVITE=YourApp: You've Been Invited
```

### App Branding

```env
STUDIO_DEFAULT_ORGANIZATION=Your Company Name
STUDIO_DEFAULT_PROJECT=Your Project Name
SMTP_SENDER_NAME=Your Company Name
```

## 🧪 Testing

### Test Email Flow

1. Access Supabase Studio at `http://localhost:3000`
2. Go to Authentication > Users
3. Click \"Send recovery email\" for a test user
4. Check email for deep link format: `yourapp:///reset-password-landing?token=...`

### Test Deep Links

1. Send password reset from your mobile app
2. Check received email
3. Tap the button/link in email
4. Verify your app opens with correct token

### Validate Configuration

```bash
# Check all services are running
docker-compose ps

# Test SMTP connection
telnet your-smtp-host 587

# Check email template service
curl http://localhost:8080/recovery.html
```

## 🚨 Security Checklist

- [ ] Strong passwords generated for all services
- [ ] JWT secrets are 32+ characters
- [ ] `.env` file not committed to Git
- [ ] SMTP credentials secured
- [ ] SSL/TLS enabled for production
- [ ] Database backups configured
- [ ] Firewall rules in place
- [ ] Deep link URLs validated in production

## 📁 File Structure

```
docker/
├── README.md                    # This file
├── SETUP_GUIDE.md              # Detailed setup instructions
├── .env.example                # Configuration template
├── docker-compose-coolify.yml  # Coolify deployment
├── docker-compose.yml          # Standard deployment
└── mail-template/
    ├── TEMPLATE_README.md      # Template documentation
    ├── confirmation.html       # Account verification email
    ├── recovery.html           # Password reset email
    ├── invite.html            # User invitation email
    ├── magic_link.html        # Magic link login email
    ├── email_change.html      # Email change confirmation
    └── reauthentication.html  # Re-authentication email
```

## 🔗 Links

- **Repository**: [GitHub](https://github.com/ykarateke/Supabase-Self-Hosted-with-Mobile-Deep-Linking)
- **Issues & Support**: [GitHub Issues](https://github.com/ykarateke/Supabase-Self-Hosted-with-Mobile-Deep-Linking/issues)
- [Detailed Setup Guide](./SETUP_GUIDE.md)
- [Email Template Documentation](./mail-template/TEMPLATE_README.md)
- [Supabase Documentation](https://supabase.com/docs)
- [Coolify Documentation](https://coolify.io/docs)

## 🆘 Troubleshooting

### Common Issues

**Email not sending**
```bash
# Check auth service logs
docker-compose logs supabase-auth

# Verify SMTP settings
# Test SMTP connection manually
```

**Deep links not working**
- Verify mobile app URL scheme configuration
- Check `GOTRUE_URI_ALLOW_LIST` in `.env`
- Validate intent filters (Android) or URL schemes (iOS)

**Database connection issues**
```bash
# Check database logs
docker-compose logs supabase-db

# Test database connection
psql -h localhost -p 5432 -U postgres
```

### Getting Help

1. Check the logs: `docker-compose logs`
2. Review [SETUP_GUIDE.md](./SETUP_GUIDE.md)
3. Open an issue: [Project Issues](https://github.com/ykarateke/Supabase-Self-Hosted-with-Mobile-Deep-Linking/issues)
4. Check [Supabase GitHub Issues](https://github.com/supabase/supabase/issues)

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request. For major changes, please open an issue first to discuss what you would like to change.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This configuration is provided as-is for educational and production use. Make sure to comply with Supabase's licensing terms.

## ⭐ Show Your Support

If this project helped you, please consider giving it a ⭐ on [GitHub](https://github.com/ykarateke/Supabase-Self-Hosted-with-Mobile-Deep-Linking)!

---

**⚠️ Production Note**: Always use SSL/TLS, secure your secrets, and implement proper monitoring in production environments.