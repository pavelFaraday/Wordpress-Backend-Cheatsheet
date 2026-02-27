# WordPress Security - Senior Interview Guide

> Security is layered architecture.
> Protect input. Protect permissions. Protect output. Protect infrastructure.

###### 1️⃣ Authentication

###### 2️⃣ Authorization

###### 3️⃣ Roles & Capabilities

###### 4️⃣ Nonce (CSRF Protection)

###### 5️⃣ CSRF

###### 6️⃣ XSS

###### 7️⃣ Sanitization

###### 8️⃣ Escaping

###### 9️⃣ SQL Injection

###### 1️⃣0️⃣ Data Validation

###### 1️⃣1️⃣ File Upload Security

###### 1️⃣2️⃣ Hardening

###### 1️⃣3️⃣ Brute Force Protection

###### 1️⃣4️⃣ REST API Security

###### 1️⃣5️⃣ Capability Checks

###### 1️⃣6️⃣ Secure AJAX

###### 1️⃣7️⃣ HTTPS & SSL

###### 1️⃣8️⃣ Password Hashing

###### 1️⃣9️⃣ Secure Cookies

###### 2️⃣0️⃣ Session Management

###### 2️⃣1️⃣ Principle of Least Privilege

###### 2️⃣2️⃣ Security Headers

###### 2️⃣3️⃣ Error Handling

###### 2️⃣4️⃣ WAF & Security Plugins

###### 2️⃣5️⃣ Secure Coding Standards

###### 2️⃣6️⃣ Update Safety

###### 2️⃣7️⃣ Backup Strategy

###### 2️⃣8️⃣ Database Security

---

# 🔐 1️⃣ Authentication

**What it means:**  
Verifying the identity of a user.

**Where in WordPress:**

- `wp-login.php`
- REST API authentication
- Application Passwords
- Auth cookies

**How to Implement:**

- Use WordPress native authentication
- Enforce HTTPS
- Use secure cookies
- Use Application Passwords or JWT for API access
- Avoid custom authentication logic

---

# 🔐 2️⃣ Authorization

**What it means:**  
Determining what an authenticated user is allowed to do.

**Where in WordPress:**

- `current_user_can()`
- Admin pages
- REST routes
- AJAX actions

**How to Implement:**

- Always check capabilities
- Restrict admin access properly
- Use `permission_callback` in REST
- Deny access by default

---

# 🔐 3️⃣ Roles & Capabilities

**What it means:**  
Role defines user type.  
Capability defines permission.

**Where in WordPress:**

- Administrator, Editor, Author
- Custom roles
- Role management APIs

**How to Implement:**

- Create roles with `add_role()`
- Assign granular capabilities
- Never check role directly
- Apply least privilege principle

---

# 🔐 4️⃣ Nonce (CSRF Protection)

**What it means:**  
Prevents unauthorized request execution.

**Where in WordPress:**

- Forms
- AJAX
- Admin actions
- REST (cookie-based)

**How to Implement:**

- Generate nonce in form/script
- Verify before processing
- Use `check_ajax_referer()`
- Reject invalid requests

---

# 🔐 5️⃣ CSRF

**What it means:**  
User performs unintended action via malicious request.

**Where in WordPress:**

- Admin actions
- Form submissions
- AJAX requests

**How to Implement:**

- Always verify nonce
- Require POST for state changes
- Validate capability

---

# 🔐 6️⃣ XSS

**What it means:**  
Malicious script injection into page.

**Where in WordPress:**

- Post content
- Comments
- Meta fields
- Templates

**How to Implement:**

- Escape all output
- Use context-aware escaping
- Limit allowed HTML with `wp_kses()`

---

# 🔐 7️⃣ Sanitization

**What it means:**  
Cleaning user input before saving.

**Where in WordPress:**

- Forms
- REST input
- Settings API
- Meta boxes

**How to Implement:**

- Sanitize before database insert
- Use appropriate sanitization function per data type
- Validate REST arguments

---

# 🔐 8️⃣ Escaping

**What it means:**  
Cleaning data before rendering.

**Where in WordPress:**

- Templates
- Admin UI
- AJAX responses
- REST responses

**How to Implement:**

- Escape before output
- Use correct escaping per context
- Never trust stored data

---

# 🔐 9️⃣ SQL Injection

**What it means:**  
Injecting malicious SQL into queries.

**Where in WordPress:**

- Custom DB queries
- `$wpdb` usage

**How to Implement:**

- Use `$wpdb->prepare()`
- Prefer WP_Query and APIs
- Avoid direct raw SQL

---

# 🔐 1️⃣0️⃣ Data Validation

**What it means:**  
Ensuring correct format and structure.

**Where in WordPress:**

- REST routes
- Form submissions
- Settings API

**How to Implement:**

- Validate before save
- Use REST `validate_callback`
- Reject invalid data

---

# 🔐 1️⃣1️⃣ File Upload Security

**What it means:**  
Preventing malicious file uploads.

**Where in WordPress:**

- Media library
- Custom upload forms

**How to Implement:**

- Use `wp_handle_upload()`
- Validate MIME types
- Disable PHP execution in uploads
- Restrict permissions

---

# 🔐 1️⃣2️⃣ Hardening

**What it means:**  
Reducing attack surface.

**Where in WordPress:**

- `wp-config.php`
- Server config
- Admin panel

**How to Implement:**

- Disable file editing
- Disable XML-RPC if unused
- Hide version
- Add security headers
- Force HTTPS

---

# 🔐 1️⃣3️⃣ Brute Force Protection

**What it means:**  
Repeated login attempts.

**Where in WordPress:**

- Login page
- REST authentication

**How to Implement:**

- Limit login attempts
- Add CAPTCHA
- Enable rate limiting
- Use WAF

---

# 🔐 1️⃣4️⃣ REST API Security

**What it means:**  
Protecting API endpoints.

**Where in WordPress:**

- Custom REST routes
- Default WP REST routes

**How to Implement:**

- Define `permission_callback`
- Authenticate properly
- Sanitize input
- Escape output

---

# 🔐 1️⃣5️⃣ Capability Checks

**What it means:**  
Verifying user permissions.

**Where in WordPress:**

- Admin pages
- AJAX handlers
- REST endpoints

**How to Implement:**

- Use `current_user_can()`
- Deny access by default
- Check before executing logic

---

# 🔐 1️⃣6️⃣ Secure AJAX

**What it means:**  
Protecting asynchronous actions.

**Where in WordPress:**

- `wp_ajax_`
- `wp_ajax_nopriv_`

**How to Implement:**

- Verify nonce
- Check capability
- Sanitize input
- Return safe response

---

# 🔐 1️⃣7️⃣ HTTPS & SSL

**What it means:**  
Encrypted communication.

**Where in WordPress:**

- Login
- Admin
- REST
- API calls

**How to Implement:**

- Install SSL
- Force HTTPS
- Secure cookies

---

# 🔐 1️⃣8️⃣ Password Hashing

**What it means:**  
Protecting stored passwords.

**Where in WordPress:**

- User authentication system

**How to Implement:**

- Use built-in auth system
- Never store plain text passwords
- Allow WP to handle hashing

---

# 🔐 1️⃣9️⃣ Secure Cookies

**What it means:**  
Protecting session data.

**Where in WordPress:**

- Authentication cookies

**How to Implement:**

- Enable Secure flag
- Enable HttpOnly
- Use SameSite policy

---

# 🔐 2️⃣0️⃣ Session Management

**What it means:**  
Managing login sessions.

**Where in WordPress:**

- Auth cookies
- Session tokens

**How to Implement:**

- Use native session handling
- Invalidate on logout
- Force password reset when needed

---

# 🔐 2️⃣1️⃣ Principle of Least Privilege

**What it means:**  
Grant minimal required access.

**Where in WordPress:**

- Custom roles
- Admin pages
- REST routes

**How to Implement:**

- Assign minimal capabilities
- Restrict sensitive actions
- Review permissions regularly

---

# 🔐 2️⃣2️⃣ Security Headers

**What it means:**  
Protecting browser communication.

**Where in WordPress:**

- Server configuration
- Security plugins

**How to Implement:**

- Add CSP
- Add HSTS
- Add X-Frame-Options
- Configure at server level

---

# 🔐 2️⃣3️⃣ Error Handling

**What it means:**  
Preventing sensitive data exposure.

**Where in WordPress:**

- REST responses
- Admin errors
- PHP errors

**How to Implement:**

- Disable debug in production
- Use `WP_Error`
- Never expose file paths or SQL errors

---

# 🔐 2️⃣4️⃣ WAF & Security Plugins

**What it means:**  
External protection layer.

**Where in WordPress:**

- Firewall plugins
- Server-level WAF

**How to Implement:**

- Enable firewall
- Enable malware scanning
- Activate rate limiting

---

# 🔐 2️⃣5️⃣ Secure Coding Standards

**What it means:**  
Writing secure code by design.

**Where in WordPress:**

- Plugin development
- Theme development

**How to Implement:**

- Follow WP coding standards
- Sanitize, validate, escape
- Use prepared statements

---

# 🔐 2️⃣6️⃣ Update Safety

**What it means:**  
Preventing vulnerabilities via outdated software.

**Where in WordPress:**

- Core
- Plugins
- Themes

**How to Implement:**

- Keep updated
- Use staging before production
- Avoid modifying core files

---

# 🔐 2️⃣7️⃣ Backup Strategy

**What it means:**  
Recovery plan.

**Where in WordPress:**

- Database
- Uploads
- Core files

**How to Implement:**

- Automated backups
- Offsite storage
- Test restore process

---

# 🔐 2️⃣8️⃣ Database Security

**What it means:**  
Protecting database access.

**Where in WordPress:**

- MySQL configuration
- wp-config.php

**How to Implement:**

- Strong DB credentials
- Restrict privileges
- Disable remote DB access
- Use firewall

---
