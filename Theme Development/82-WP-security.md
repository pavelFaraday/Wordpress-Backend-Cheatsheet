# WordPress Security

🗣️ **Senior WordPress Security Interview Preparation** 🗣️

---

# 🔐 1️⃣ Authentication

**What it means:**
Verifying who the user is.

**Where in WordPress:**

- Login system (`wp-login.php`)
- REST API authentication
- Application Passwords
- Cookies
- JWT plugins

---

# 🔐 2️⃣ Authorization

**What it means:**
What the user is allowed to do.

**Where in WordPress:**

- `current_user_can()`
- Roles & Capabilities
- `permission_callback` in REST
- Admin page restrictions

---

# 🔐 3️⃣ Roles & Capabilities

**What it means:**
Role = user type
Capability = specific permission

**Where in WordPress:**

- Administrator
- Editor
- Author
- Custom roles
- `add_role()`
- `add_cap()`

Senior note:
Never check role directly. Always check capability.

---

# 🔐 4️⃣ Nonce (Number used once)

**What it means:**
Protection against CSRF attacks.

**Where in WordPress:**

- Forms
- Admin actions
- AJAX
- REST (cookie-based auth)

Functions:

- `wp_create_nonce()`
- `wp_verify_nonce()`
- `check_ajax_referer()`

---

# 🔐 5️⃣ CSRF (Cross-Site Request Forgery)

**What it means:**
Attacker tricks logged-in user into performing an action.

**Where protected in WordPress:**

- Nonces
- Admin actions
- Form submissions

---

# 🔐 6️⃣ XSS (Cross-Site Scripting)

**What it means:**
Malicious JavaScript injected into site.

**Where prevented in WordPress:**

- Output escaping
- Comment system
- Post content
- Meta fields

Functions:

- `esc_html()`
- `esc_attr()`
- `esc_url()`
- `wp_kses()`

---

# 🔐 7️⃣ Sanitization

**What it means:**
Cleaning user input before saving.

**Where used:**

- Form inputs
- REST API input
- Settings API
- Meta boxes

Functions:

- `sanitize_text_field()`
- `sanitize_email()`
- `sanitize_key()`
- `absint()`

---

# 🔐 8️⃣ Escaping

**What it means:**
Cleaning output before displaying.

**Where used:**

- Templates
- Admin pages
- AJAX responses
- REST responses

Senior rule:
Sanitize before save. Escape before output.

---

# 🔐 9️⃣ SQL Injection

**What it means:**
Attacker injects SQL into database query.

**Where prevented in WordPress:**

- `$wpdb->prepare()`
- Using WP_Query safely
- Avoiding raw SQL

Example:

```php
$wpdb->prepare("SELECT * FROM $table WHERE id = %d", $id);
```

---

# 🔐 1️⃣0️⃣ Data Validation

**What it means:**
Checking if data is correct format.

**Where used:**

- REST API
- Form submission
- Settings API
- Custom endpoints

REST example:

```php
'args' => [
   'email' => [
      'validate_callback' => 'is_email'
   ]
]
```

---

# 🔐 1️⃣1️⃣ File Upload Security

**What it means:**
Prevent malicious files.

**Where in WordPress:**

- Media upload
- Custom upload forms

Security:

- `wp_handle_upload()`
- MIME type checks
- Disable PHP execution in uploads folder

---

# 🔐 1️⃣2️⃣ Hardening

**What it means:**
Reducing attack surface.

**Where in WordPress:**

- Disable XML-RPC
- Hide WP version
- Disable file editing
- Force HTTPS
- Security headers

---

# 🔐 1️⃣3️⃣ XML-RPC

**What it means:**
Remote communication protocol.

**Security issue:**
Brute-force attacks.

Where:

- `xmlrpc.php`

Often disabled if unused.

---

# 🔐 1️⃣4️⃣ Brute Force Protection

**What it means:**
Repeated login attempts.

Where:

- Login page
- REST auth

Protection:

- Limit login attempts
- CAPTCHA
- Rate limiting
- WAF

---

# 🔐 1️⃣5️⃣ REST API Security

Where security applies:

- `permission_callback`
- Authentication methods
- Input validation
- Output sanitization

Never:

```php
'permission_callback' => '__return_true'
```

---

# 🔐 1️⃣6️⃣ Capability Checks

Used in:

- Admin pages
- REST endpoints
- AJAX actions

Example:

```php
current_user_can('manage_options')
```

---

# 🔐 1️⃣7️⃣ Secure AJAX

Where:

- `wp_ajax_`
- `wp_ajax_nopriv_`

Security:

- Nonce verification
- Capability checks
- Sanitization

---

# 🔐 1️⃣8️⃣ HTTPS & SSL

Where:

- Admin
- Login
- API communication

Forces encrypted communication.

---

# 🔐 1️⃣9️⃣ Password Hashing

WordPress uses:

- `wp_hash_password()`
- PHP password hashing internally

Passwords never stored in plain text.

---

# 🔐 2️⃣0️⃣ Secure Cookies

Where:

- Authentication
- Login sessions

Flags:

- HttpOnly
- Secure
- SameSite

---

# 🔐 2️⃣1️⃣ Session Management

WordPress uses:

- Auth cookies
- Token validation
- Session invalidation

---

# 🔐 2️⃣2️⃣ Principle of Least Privilege

Meaning:
Give minimum access required.

Where:

- Custom roles
- REST endpoints
- Admin pages

---

# 🔐 2️⃣3️⃣ Security Headers

Examples:

- Content-Security-Policy
- X-Frame-Options
- X-XSS-Protection
- Strict-Transport-Security

Configured via server or plugin.

---

# 🔐 2️⃣4️⃣ Cross-Origin Resource Sharing (CORS)

Where:

- REST API
- Headless setups

Controls who can access your API.

---

# 🔐 2️⃣5️⃣ Error Handling

Never expose:

- SQL errors
- File paths
- Stack traces

Use:
`WP_Error`

---

# 🔐 2️⃣6️⃣ Security Plugins & WAF

Examples:

- Firewall
- Malware scanning
- Rate limiting

Protection layer outside WordPress.

---

# 🔐 2️⃣7️⃣ Secure Coding Standards

Follow:

- WordPress Coding Standards
- Input validation
- Escaping rules
- Prepared statements

---

# 🔐 2️⃣8️⃣ Update Safety

Always:

- Keep core updated
- Keep plugins updated
- Avoid modifying core

---

# 🔐 2️⃣9️⃣ Backup Strategy

Security includes:

- Regular backups
- Offsite storage
- Database backups

---

# 🔐 3️⃣0️⃣ Database Security

- Proper DB user privileges
- Strong DB password
- Table prefix change (basic hardening)
- Disable remote DB access

---

# 🎯 What Makes You Senior in Security?

You:

- Think in attack vectors
- Think in validation layers
- Think in permissions
- Think in architecture
- Secure REST, AJAX, Forms, Admin, Database

Security is not a feature.
It is a mindset.

---
