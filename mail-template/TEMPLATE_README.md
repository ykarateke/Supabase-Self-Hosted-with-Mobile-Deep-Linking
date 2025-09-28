# 📧 Email Templates Documentation

This directory contains custom HTML email templates for Supabase authentication flows with mobile deep linking support.

## 📁 Template Files

| Template | Purpose | Deep Link Path |
|----------|---------|----------------|
| `confirmation.html` | Account verification after signup | `/auth-callback` |
| `recovery.html` | Password reset emails | `/reset-password-landing` |
| `invite.html` | User invitation emails | `/auth-callback` |
| `magic_link.html` | Passwordless login links | `/auth-callback` |
| `email_change.html` | Email address change confirmation | `/auth-callback` |
| `reauthentication.html` | Re-authentication requests | `/auth-callback` |

## 🔗 Deep Link Structure

All templates generate deep links in this format:
```
yourapp:///path?token={TokenHash}&type={type}
```

### Examples:
- **Password Reset**: `myapp:///reset-password-landing?token=abc123&type=recovery`
- **Email Confirmation**: `myapp:///auth-callback?token=def456&type=signup`
- **Magic Link**: `myapp:///auth-callback?token=ghi789&type=magiclink`

## 🛠️ Installation

### For Coolify Deployment

1. Copy all template files to your Coolify service volume:
```bash
# In Coolify file manager, create folder structure:
/data/coolify/services/your-service-id/volumes/email-templates/
```

2. Upload template files to this directory:
```
email-templates/
├── confirmation.html
├── recovery.html
├── invite.html
├── magic_link.html
├── email_change.html
└── reauthentication.html
```

### For Standard Docker Compose

1. Create local directory structure:
```bash
mkdir -p ./volumes/email-templates
```

2. Copy templates to the directory:
```bash
cp mail-template/*.html ./volumes/email-templates/
```

3. Verify volume mount in `docker-compose.yml`:
```yaml
email-templates:
  image: 'nginx:alpine'
  volumes:
    - ./volumes/email-templates:/usr/share/nginx/html:ro
```

## 🎨 Customization Guide

### 1. Branding & Colors

Update these elements in each template:

#### Logo
```html
<!-- Find this line and update src -->
<img alt=\"Your App Logo\" src=\"https://yourdomain.com/logo.png\" />
```

#### Primary Color
```html
<!-- Update background color -->
<td bgcolor=\"#CAE9D5\">  <!-- Change this hex color -->
```

#### Company Name
```html
<!-- Update footer -->
<div>© 2024 Your Company Name. All rights reserved.</div>
```

### 2. Text Content

#### Email Subjects
Configure in `.env` file:
```env
MAILER_SUBJECTS_CONFIRMATION=YourApp: Verify Your Account
MAILER_SUBJECTS_RECOVERY=YourApp: Reset Your Password
MAILER_SUBJECTS_INVITE=YourApp: You've Been Invited
MAILER_SUBJECTS_MAGIC_LINK=YourApp: Login to Your Account
MAILER_SUBJECTS_EMAIL_CHANGE=YourApp: Confirm Email Change
```

#### Button Text
Find and customize button labels:
```html
<a href=\"...\">Your Custom Button Text</a>
```

#### Body Content
Update the main message text in each template.

### 3. Deep Link URLs

The templates use these variables to construct deep links:

- `{{.TokenHash}}` - Authentication token
- `{{.Type}}` - Authentication type (recovery, signup, etc.)

Current URL structure:
```html
<a href=\"yourapp:///path?token={{.TokenHash}}&type=recovery\">
```

To change the app scheme, update each template:
```html
<!-- From -->
<a href=\"yourapp:///reset-password-landing?token={{.TokenHash}}&type=recovery\">

<!-- To -->
<a href=\"myawesomeapp:///reset-password-landing?token={{.TokenHash}}&type=recovery\">
```

## 🔧 Template Variables

### Available Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `{{.TokenHash}}` | Hashed authentication token | `pkce_abc123def456` |
| `{{.Token}}` | Raw authentication token | `def456ghi789` |
| `{{.Type}}` | Authentication type | `recovery`, `signup`, `invite` |
| `{{.Email}}` | User's email address | `user@example.com` |
| `{{.SiteURL}}` | Site URL from configuration | `https://yoursite.com` |

### Usage Examples

```html
<!-- Deep link with token -->
<a href=\"myapp:///auth?token={{.TokenHash}}&type={{.Type}}\">Verify</a>

<!-- Display user email -->
<p>This email was sent to {{.Email}}</p>

<!-- Fallback URL -->
<p>If the button doesn't work, visit: {{.SiteURL}}</p>
```

## 🧪 Testing Templates

### 1. Template Accessibility Test
```bash
# Test if templates are accessible
curl http://localhost:8080/recovery.html
curl http://localhost:8080/confirmation.html

# Should return HTML content
```

### 2. Email Sending Test

1. Access Supabase Studio: `http://localhost:3000`
2. Go to **Authentication** → **Users**
3. Select a user and click **"Send recovery email"**
4. Check email for:
   - Proper HTML formatting
   - Correct deep link format
   - Working button/link

### 3. Deep Link Test

Send test email and verify link format:
```
✅ Correct: myapp:///reset-password-landing?token=abc123&type=recovery
❌ Wrong:   https://domain.com/auth/v1/verify?token=abc123&redirect_to=...
```

## 🔄 Template Updates

### After Making Changes

1. **Restart email service**:
```bash
docker-compose restart email-templates
```

2. **Clear browser cache** if testing via browser

3. **Send test email** to verify changes

### For Coolify Users

1. Update files in Coolify file manager
2. Restart the email-templates service
3. Test email sending

## 🚨 Common Issues

### Templates Not Loading

**Problem**: Emails show broken formatting or default text

**Solutions**:
1. Check template files exist in correct directory
2. Verify volume mount in docker-compose.yml
3. Check email-templates service logs: `docker logs email-templates`
4. Test template accessibility: `curl http://localhost:8080/recovery.html`

### Deep Links Not Working

**Problem**: Links don't open mobile app

**Solutions**:
1. Verify app URL scheme configuration
2. Check intent filters (Android) or URL schemes (iOS)
3. Ensure deep link URLs in templates match app configuration
4. Test deep link manually: `adb shell am start -d \"myapp:///test\"`

### Variables Not Rendering

**Problem**: Email shows `{{.TokenHash}}` instead of actual token

**Solutions**:
1. Check GoTrue environment variables
2. Verify template syntax (use `{{.Variable}}` format)
3. Restart supabase-auth service
4. Check auth service logs for template errors

## 📱 Mobile App Integration

### Flutter Example

```dart
// Handle incoming deep links
void handleAuthLink(String link) {
  final uri = Uri.parse(link);
  final token = uri.queryParameters['token'];
  final type = uri.queryParameters['type'];

  switch (type) {
    case 'recovery':
      // Navigate to password reset screen
      Navigator.pushNamed(context, '/reset-password', arguments: token);
      break;
    case 'signup':
      // Handle signup confirmation
      await Supabase.instance.client.auth.verifyOTP(
        token: token!,
        type: OtpType.signup,
      );
      break;
  }
}
```

### React Native Example

```javascript
import { Linking } from 'react-native';
import { supabase } from './supabase';

// Listen for deep links
Linking.addEventListener('url', handleDeepLink);

function handleDeepLink({ url }) {
  const urlParts = url.split('?');
  const params = new URLSearchParams(urlParts[1]);

  const token = params.get('token');
  const type = params.get('type');

  if (type === 'recovery') {
    // Navigate to password reset screen
    navigation.navigate('ResetPassword', { token });
  }
}
```

## 🔒 Security Notes

- Templates are served read-only
- No external ports exposed for email-templates service
- Tokens are automatically hashed by GoTrue
- Links expire based on GoTrue configuration
- Always use HTTPS in production

## 📞 Support

If you encounter issues:

1. Check the [main README](../README.md)
2. Review [SETUP_GUIDE](../SETUP_GUIDE.md)
3. Open an issue: [Project Issues](https://github.com/ykarateke/Supabase-Self-Hosted-with-Mobile-Deep-Linking/issues)
4. Check Docker logs: `docker-compose logs email-templates`
5. Verify template syntax and variables
6. Test deep links on actual devices

---

**💡 Tip**: Always test email templates on multiple devices and email clients to ensure compatibility.