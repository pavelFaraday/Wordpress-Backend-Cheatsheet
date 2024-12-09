# 📌 `esc_()` family functions

In WordPress, the `esc_()` family of functions ensures that user-provided or dynamically generated data is properly escaped before being output to the browser, preventing cross-site scripting (XSS) and other security vulnerabilities. Below is a detailed list of the core `esc_()` functions, their purposes, core concepts, and practical use cases:

---

### **1. `esc_html()`**
- **Purpose:** Escapes HTML to prevent the injection of malicious HTML or JavaScript into the page.
- **Core Concept:** Converts special characters (`<`, `>`, `&`, etc.) into HTML entities to safely display user-generated content.
- **Use Cases in Practice:**
  ```php
  $user_input = "<script>alert('XSS');</script>";
  echo esc_html($user_input); // Outputs: &lt;script&gt;alert(&#039;XSS&#039;);&lt;/script&gt;
  ```
  - Use for: Outputting plain text in HTML contexts (e.g., in paragraph tags, divs, or spans).

---

### **2. `esc_attr()`**
- **Purpose:** Escapes content for use in HTML attributes (e.g., `id`, `class`, `data-*`).
- **Core Concept:** Encodes quotes and other characters that could terminate or manipulate attributes.
- **Use Cases in Practice:**
  ```php
  $class_name = "user-input\" onclick=\"alert('hack')";
  echo '<div class="' . esc_attr($class_name) . '"></div>';
  // Outputs: <div class="user-input&quot; onclick=&quot;alert(&#039;hack&#039;)"></div>
  ```
  - Use for: Dynamic data inside HTML attributes.

---

### **3. `esc_url()`**
- **Purpose:** Escapes URLs to prevent malicious links from being injected into your site.
- **Core Concept:** Validates and escapes URLs, ensuring safe output in `href`, `src`, and similar attributes.
- **Use Cases in Practice:**
  ```php
  $url = "javascript:alert('XSS');";
  echo '<a href="' . esc_url($url) . '">Click here</a>';
  // Outputs: <a href="">Click here</a>
  ```
  - Use for: Links, images, and any attributes that require URLs.

---

### **4. `esc_js()`**
- **Purpose:** Escapes strings for safe output in JavaScript contexts.
- **Core Concept:** Encodes special characters for inline JavaScript to prevent breaking scripts or injecting malicious code.
- **Use Cases in Practice:**
  ```php
  $user_input = "alert('Hello!');";
  echo '<script>var message = "' . esc_js($user_input) . '";</script>';
  // Outputs: <script>var message = "alert(\u0027Hello!\u0027);";</script>
  ```
  - Use for: Embedding dynamic data within `<script>` tags or inline JavaScript.

---

### **5. `esc_textarea()`**
- **Purpose:** Escapes content for safe use inside a `<textarea>` element.
- **Core Concept:** Ensures that input is treated as text, escaping special characters like `<` and `>`.
- **Use Cases in Practice:**
  ```php
  $user_comment = "<script>alert('Hack');</script>";
  echo '<textarea>' . esc_textarea($user_comment) . '</textarea>';
  // Outputs: <textarea>&lt;script&gt;alert(&#039;Hack&#039;);&lt;/script&gt;</textarea>
  ```
  - Use for: Displaying user-provided or dynamic content in textareas.

---

### **6. `esc_url_raw()`**
- **Purpose:** Similar to `esc_url()` but used specifically for storing or processing URLs, not directly outputting them.
- **Core Concept:** Strips invalid characters from a URL but does not encode it for HTML.
- **Use Cases in Practice:**
  ```php
  $url = "https://example.com/?q=<script>alert('XSS');</script>";
  $sanitized_url = esc_url_raw($url);
  update_option('custom_url', $sanitized_url);
  ```
  - Use for: Storing sanitized URLs in the database or passing them to APIs.

---

### **7. `esc_sql()`**
- **Purpose:** Escapes data for use in SQL queries, preventing SQL injection.
- **Core Concept:** Prepares data to be safely inserted into database queries.
- **Use Cases in Practice:**
  ```php
  global $wpdb;
  $user_input = "'; DROP TABLE wp_users; --";
  $safe_input = esc_sql($user_input);
  $query = "SELECT * FROM {$wpdb->prefix}users WHERE username = '$safe_input'";
  ```
  - Use for: Escaping values in direct SQL queries. Note: Preferred approach is to use prepared statements.

---

### **8. `wp_kses()`**
- **Purpose:** Allows only a specified set of HTML tags and attributes.
- **Core Concept:** Strips or escapes all other elements not explicitly allowed.
- **Use Cases in Practice:**
  ```php
  $content = "<b>Bold</b> <script>alert('XSS');</script>";
  $allowed_tags = array('b' => array());
  echo wp_kses($content, $allowed_tags);
  // Outputs: <b>Bold</b>
  ```
  - Use for: Sanitizing rich content like post editor input.

---

### **9. `wp_kses_post()`**
- **Purpose:** A predefined version of `wp_kses()` that allows the same HTML tags and attributes permitted in WordPress posts.
- **Core Concept:** Ideal for escaping content meant for post output.
- **Use Cases in Practice:**
  ```php
  $user_content = "<strong>Safe</strong> <script>alert('Hack');</script>";
  echo wp_kses_post($user_content);
  // Outputs: <strong>Safe</strong>
  ```
  - Use for: Sanitizing content submitted through forms that mimic post submission.

---

### Best Practices
- **Context Matters:** Always use the escaping function suited to the context of the output.
- **Combine with Validation:** Use `sanitize_*()` functions for input validation and `esc_()` functions for output escaping.
- **Default to Escaping:** When in doubt, escape all user-generated data before output.

These functions ensure that WordPress remains secure and user data is processed safely, while still allowing flexibility in designing functional and interactive sites.

---
---
---

# 📌 `sanitize_()` family functions

The `sanitize_` family of functions in WordPress is used to **sanitize user input** to ensure it meets expected data types, formats, and security standards. **While `esc_()` functions are used for escaping data during <u>output</u>, `sanitize_` functions are applied during <u>input</u> handling to clean and validate data before storing it in the database or using it elsewhere.**

Here’s a detailed breakdown of key `sanitize_` functions, their purposes, and practical use cases:

---

### **1. `sanitize_text_field()`**
- **Purpose:** Strips tags, removes invalid UTF-8 characters, and trims leading/trailing whitespace.
- **Core Concept:** Use for single-line text input that should not contain any HTML.
- **Use Cases in Practice:**
  ```php
  $user_input = "<script>alert('Hack');</script> Hello World!";
  $sanitized = sanitize_text_field($user_input);
  echo $sanitized; // Outputs: Hello World!
  ```
  - Use for: Text fields like names, usernames, or titles.

---

### **2. `sanitize_email()`**
- **Purpose:** Sanitizes email addresses by removing invalid characters.
- **Core Concept:** Verifies that the email is valid and safe for storage or further processing.
- **Use Cases in Practice:**
  ```php
  $email = "bad-email<script>@example.com";
  $sanitized_email = sanitize_email($email);
  echo $sanitized_email; // Outputs: bad-email@example.com
  ```
  - Use for: Email input fields in forms.

---

### **3. `sanitize_url()` (Deprecated)**
- **Purpose:** Deprecated in favor of `esc_url_raw()`, it sanitized URLs to remove unsafe characters.
- **Alternative:** Use `esc_url_raw()` for sanitizing URLs for storage.

---

### **4. `sanitize_file_name()`**
- **Purpose:** Removes unsafe characters from file names and converts them to safe, web-friendly strings.
- **Core Concept:** Ensures that uploaded or generated file names are secure and valid.
- **Use Cases in Practice:**
  ```php
  $file_name = "my file<script>.jpg";
  $safe_file_name = sanitize_file_name($file_name);
  echo $safe_file_name; // Outputs: my-file.jpg
  ```
  - Use for: Handling file uploads or file name generation.

---

### **5. `sanitize_key()`**
- **Purpose:** Strips out characters not allowed in WordPress option or metadata keys.
- **Core Concept:** Use for internal data like option keys or meta keys.
- **Use Cases in Practice:**
  ```php
  $key = "user<input>key";
  $sanitized_key = sanitize_key($key);
  echo $sanitized_key; // Outputs: userinputkey
  ```
  - Use for: Sanitizing array keys or option names.

---

### **6. `sanitize_title()`**
- **Purpose:** Cleans up titles for use in URLs or as slugs.
- **Core Concept:** Converts the title into a lowercase, URL-friendly slug by removing HTML and unsafe characters.
- **Use Cases in Practice:**
  ```php
  $title = "Hello <b>World</b>!";
  $slug = sanitize_title($title);
  echo $slug; // Outputs: hello-world
  ```
  - Use for: Generating post slugs or URL segments.

---

### **7. `sanitize_title_with_dashes()`**
- **Purpose:** Similar to `sanitize_title()` but focuses explicitly on replacing spaces with dashes.
- **Core Concept:** Primarily used for creating slugs from titles.
- **Use Cases in Practice:**
  ```php
  $title = "My Cool Post!";
  $slug = sanitize_title_with_dashes($title);
  echo $slug; // Outputs: my-cool-post
  ```

---

### **8. `sanitize_meta()`**
- **Purpose:** Sanitizes metadata before saving it to the database.
- **Core Concept:** Requires the metadata key and type to ensure the input is properly validated.
- **Use Cases in Practice:**
  ```php
  $value = "<script>alert('Hack');</script>";
  $sanitized_meta = sanitize_meta('custom_meta', $value, 'post');
  echo $sanitized_meta; // Outputs: alert('Hack');
  ```
  - Use for: Custom post meta, user meta, or term meta.

---

### **9. `sanitize_option()`**
- **Purpose:** Sanitizes values for WordPress options.
- **Core Concept:** Built to work with WordPress option keys and their associated validation requirements.
- **Use Cases in Practice:**
  ```php
  $value = "<script>alert('Hack');</script>";
  $sanitized_option = sanitize_option('blogname', $value);
  echo $sanitized_option; // Outputs: alert('Hack');
  ```
  - Use for: Validating or updating site options.

---

### **10. `sanitize_html_class()`**
- **Purpose:** Sanitizes strings for use as CSS class names.
- **Core Concept:** Removes invalid characters and ensures the class name complies with HTML and CSS standards.
- **Use Cases in Practice:**
  ```php
  $class_name = "invalid-class<script>";
  $sanitized_class = sanitize_html_class($class_name);
  echo $sanitized_class; // Outputs: invalid-class
  ```
  - Use for: Dynamic class names in HTML.

---

### **11. `sanitize_textarea_field()`**
- **Purpose:** Similar to `sanitize_text_field()` but designed for multi-line text input.
- **Core Concept:** Strips tags, invalid UTF-8, and ensures safe multi-line content.
- **Use Cases in Practice:**
  ```php
  $textarea = "<b>Bold</b>\n<script>alert('Hack');</script>";
  $sanitized_textarea = sanitize_textarea_field($textarea);
  echo $sanitized_textarea; 
  // Outputs: Bold
  //          alert('Hack');
  ```
  - Use for: Textarea input in forms.

---

### **12. `sanitize_user()`**
- **Purpose:** Sanitizes usernames by removing unsafe characters.
- **Core Concept:** Ensures that usernames are valid and safe for the WordPress user system.
- **Use Cases in Practice:**
  ```php
  $username = "User<script>123";
  $sanitized_user = sanitize_user($username);
  echo $sanitized_user; // Outputs: User123
  ```
  - Use for: Registration or username updates.

---

### **13. `wp_filter_nohtml_kses()`**
- **Purpose:** Removes all HTML from input.
- **Core Concept:** Strips all tags and keeps only plain text.
- **Use Cases in Practice:**
  ```php
  $input = "<b>Bold</b> <script>alert('Hack');</script>";
  $sanitized = wp_filter_nohtml_kses($input);
  echo $sanitized; // Outputs: Bold alert('Hack');
  ```
  - Use for: Enforcing strict plain-text input.

---

### **Best Practices**
1. **Choose the Right Sanitizer:** Each function serves specific input types and contexts. Ensure the sanitizer matches the expected format.
2. **Combine with Escaping:** Sanitize input during processing and escape output during rendering.
3. **Validate Input:** Use `sanitize_` functions alongside validation (e.g., `filter_var` for emails or URLs).

---

By using the `sanitize_` family appropriately, you can protect your WordPress application from malicious input and ensure data integrity across your site.