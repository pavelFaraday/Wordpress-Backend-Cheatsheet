# Security & Sanitization in Gutenberg Block Development (WP 6+)

Locking down blocks is non-negotiable: your editor runs untrusted content, renders on both admin and front-end, and often touches REST, meta, and SSR. Below is a practical, opinionated guide with patterns you can copy.

---

## Core principles (mental model)

1. **❗️Sanitize on input, validate in logic, escape on output.**

   - **Sanitize** = clean/strip unsafe pieces before storing/processing (`sanitize_text_field`, `absint`, `esc_url_raw`, etc.).
     - 👉 Example: If a user types `<script>bad()</script>`, save only the text part, not the script.
   - **Validate** = Check the data follows your/business rules
     - 👉 Example: If you expect a number `0–100`, make sure it’s in that range.
   - **Escape** = Protect the data when showing it, encode for the _context you’re outputting into_ (`esc_html`, `esc_attr`, `esc_url`, etc.).
     - 👉 Example: Use `esc_html()` so text doesn’t run as code in the browser.

>

2. **❗️Prefer server-side guarantees.**
   Don’t rely only on JavaScript for safety.
   - Always secure your data in PHP when registering meta or rendering blocks.
   - Register meta with built-in functions like **`sanitize_callback`** + `type` + `auth_callback`.
   - In dynamic blocks (PHP `render_callback`), **escape every variable** by context.
     - 👉 Example: Even if someone bypasses the editor, the server will still block bad data.

>

3. **❗️React is safe by default—until you bypass it.**

   - JSX escapes text nodes automatically. So user text is safe..
   - Risks appear when you use **`dangerouslySetInnerHTML`**, inject inline styles/HTML from user input, or interpolate strings into `style`/`className` without guardrails.
     - 👉 Rule: Only use raw HTML if you really know it’s safe.

>

4. **❗️Least privilege everywhere.**

   - Only give features that are actually needed.
   - Don’t let users add random HTML, custom styles, or classes unless your block requires it.
   - For REST API, always provide a **`permission_callback`**, check permissions and nonces before saving data.

>

5. **✅ Simple takeaway:**
   - Clean on save, check rules, escape on show.
   - Trust PHP/server more than JS.
   - Don’t open doors you don’t need.

---

## Static vs Dynamic blocks (where to secure)

### Static Blocks

- **What they are:**
  A static block saves its **HTML directly into the post content** when you hit “Update.”
  👉 Example: A simple paragraph block saves `<p>Hello World</p>` right into the database.

- **Where security matters:**

  - WordPress already runs **content filters/kses** on post content (depending on user permissions).
  - BUT you should still be careful! Don’t save dangerous things like:

    - `<script>` tags
    - inline JavaScript (`onclick="..."`)

  - Instead of storing raw inline styles from users (e.g., `style="color:red;font-size:30px"`), store **safe tokens or class names** (`class="has-red-text"`).

👉 **Simple rule for static blocks:**
“Save only clean, safe HTML. Use classes or tokens instead of raw inline styles.”

### Dynamic Blocks

- **What they are:**
  A dynamic block doesn’t save full HTML into post content. Instead, it saves a **placeholder**, and the final HTML is generated **every time the page loads** with a `render_callback` in PHP.
  👉 Example: A Latest Posts block — it only stores “settings” in the post, and PHP builds the list of posts on the front end.

- **Where security matters:**

  - Because you’re generating HTML dynamically on the server, **you are responsible** for escaping every attribute and piece of content you output.
  - Treat everything from `$attributes` as **untrusted**.
    👉 Example: If `$attributes['title']` comes from a user, escape it with `esc_html()` before printing.

👉 **Simple rule for dynamic blocks:**
“Always escape everything you print, because the server creates the HTML fresh each time.”

### ✅ Simple akeaway

- 📌 **Static block** → Saves its HTML once. Keep it simple, safe, and clean (use classes, not risky inline code).
- 📌 **Dynamic block** → Renders fresh HTML every time. Escape **every variable** you output.

---

## Escaping & sanitizing block attributes (PHP)

### Common sanitizers (before storing / in logic)

- Text: `sanitize_text_field()`, `wp_strip_all_tags()`
- Integers: `absint()`; numbers: `(float)`, `floatval()`
- Booleans: `(bool)`
- URLs going **into storage**: `esc_url_raw()`
- Hex colors: `sanitize_hex_color()`, `sanitize_hex_color_no_hash()`
- Emails: `sanitize_email()`
- Slugs/keys: `sanitize_key()`, `sanitize_title()`
- Arrays: validate shape; recursively sanitize items

### Common escapers (when outputting)

- HTML text nodes: `esc_html( $text )`
- HTML attributes: `esc_attr( $value )`
- URLs in `href`/`src`: `esc_url( $url )`
- `<textarea>` contents: `esc_textarea( $text )`
- JSON inline to attributes: `esc_attr( wp_json_encode( $data ) )`
- JS strings (rarely in blocks): `esc_js( $string )` (prefer avoiding inline JS)

> **Rule of thumb:** sanitize early (on save or when receiving), escape late (right before echoing in the right context).

---

## Avoiding XSS with `dangerouslySetInnerHTML` (JS)

- **Default:** Don’t use it. React escapes text for you.
- **If you must** (e.g., rendering trusted server HTML in the editor preview):

  - Only feed **trusted, pre-sanitized** HTML (e.g., from your PHP `render_callback`).
  - Never pass **user-provided raw HTML** directly.
  - Prefer Gutenberg primitives like **`<RichText>`** for user-editable content and generate final markup server-side.
  - For simple formatting in editor UI, use components (TextControl, Notice, PanelBody) instead of raw HTML injection.

---

## Registering meta and REST safely

- **Post/meta registration**:

  ```php
  register_post_meta( 'post', '_my_key', [
    'type'              => 'string',
    'single'            => true,
    'show_in_rest'      => true,
    'sanitize_callback' => 'sanitize_text_field',
    'auth_callback'     => fn() => current_user_can( 'edit_posts' ),
  ] );
  ```

  This ensures anything saved through the editor/REST is sanitized and access-controlled.

- **Custom REST routes**:

  ```php
  register_rest_route( 'ns/v1', '/thing', [
    'methods'  => 'POST',
    'callback' => 'ns_do_thing',
    'permission_callback' => function( $request ) {
      return current_user_can( 'edit_posts' ) && wp_verify_nonce( $request->get_header( 'X-WP-Nonce' ), 'wp_rest' );
    },
    'args'     => [
      'count' => [ 'type' => 'integer', 'sanitize_callback' => 'absint' ],
      'url'   => [ 'type' => 'string',  'sanitize_callback' => 'esc_url_raw' ],
    ],
  ] );
  ```

  Always provide **`permission_callback`** and validate/sanitize **`args`**.

---

## Safe rendering in `render_callback` (dynamic blocks)

- **Whitelist attributes** you expect; coerce types with `wp_parse_args` and casts.

- Escape all variables **by context**:

  ```php
  $title = isset( $attributes['title'] ) ? sanitize_text_field( $attributes['title'] ) : '';
  $url   = isset( $attributes['url'] )   ? esc_url( $attributes['url'] )           : '#';

  printf(
    '<a class="my-cta" href="%s"><span>%s</span></a>',
    esc_url( $url ),
    esc_html( $title )
  );
  ```

- **Inline styles:** Avoid printing arbitrary `style=""` from user input. If you must, restrict to **numeric/known tokens** (e.g., spacing scale), validate each value, then escape with `esc_attr`.

- **HTML fragments from elsewhere:** Run through `wp_kses()` with an allowlist (`wp_kses_allowed_html( 'post' )` or custom).

---

## Database & queries

- **Never** concatenate SQL. Use `$wpdb->prepare()` for values.
- `esc_sql()` is not a substitute for prepared statements (use it only for wildcards in LIKE).
- Validate numeric IDs with `absint()` **before** using them in any query.

---

## Media & files

- Validate file types with `wp_check_filetype_and_ext()`.
- Use the upload APIs (`media_handle_upload`, `wp_handle_upload`) and respect capability checks.
- Escape attachment URLs with `esc_url`. Don’t trust MIME sent by the client.

---

## Block supports, theme tokens, and safety

- Disable what you don’t need in **`supports`** (e.g., `"html": false`, limit `"color"`, `"spacing"`, `"typography"`). The less is exposed, the less can be abused.
- Use **`theme.json` tokens** (preset colors, spacing) instead of accepting arbitrary values. This converts potentially unsafe freeform inputs into **controlled presets**.

---

## Editor UI patterns that reduce risk

- Prefer **controlled inputs** (TextControl, URLInput, Notice, ToggleControl) over raw `<input>` with custom handlers.
- Validate on change (e.g., check `isURL`, `sanitize_hex_color`) and provide immediate feedback.
- Keep dangerous operations behind clear user intent (confirmations, capabilities).

---

## ❗️❗️❗️ Quick cheat-sheet (pasteable)

**Input (sanitize/validate)**

- string → `sanitize_text_field()`
- url → `esc_url_raw()` (store), `esc_url()` (output)
- int → `absint()`
- color → `sanitize_hex_color()`
- email → `sanitize_email()`
- arrays → validate shape; sanitize each item
- HTML → `wp_kses( $html, $allowed_tags )`

**Output (escape)**

- text node → `esc_html()`
- attribute → `esc_attr()`
- href/src → `esc_url()`
- textarea → `esc_textarea()`
- json in data-attr → `esc_attr( wp_json_encode( $data ) )`

**JS (editor)**

- Avoid `dangerouslySetInnerHTML`. If unavoidable, only with **trusted/sanitized** HTML.
- Use `<RichText>` for user text; React will escape values.

**REST/meta**

- Always set `permission_callback`.
- Add `sanitize_callback` for meta/args.
- Verify **`X-WP-Nonce`** for state changes.

**Queries**

- `$wpdb->prepare()`; never raw concatenation.

---

## High-value use cases

- **Dynamic listing block**: attributes (count, taxonomy) are sanitized (`absint`, `sanitize_key`), query uses prepared statements/`WP_Query`, output escaped per context.
- **CTA block**: URL validated with `esc_url_raw` on save; front-end uses `esc_url` + `esc_html` for text.
- **User-pasted HTML** (advanced block): run through `wp_kses_post()` and **never** render it with `dangerouslySetInnerHTML` unless already sanitized.

---

## Interview-style talking points (with strong answers)

**📌 Q1. Outline your security model for Gutenberg blocks.**
**A.** I follow **sanitize → validate → escape**. Inputs (REST/meta/attributes) are sanitized on receipt; logic enforces constraints; rendering escapes values per context. I minimize attack surface by restricting block supports and using theme tokens, and I avoid `dangerouslySetInnerHTML`—if used, only with trusted, already-sanitized HTML.

**📌 Q2. How do you prevent XSS in a dynamic block’s `render_callback`?**
**A.** Treat `$attributes` as untrusted, coerce types (`absint`, `sanitize_text_field`, `esc_url_raw`), and escape output (`esc_html`, `esc_attr`, `esc_url`) at the exact context. For any HTML fragments, I use `wp_kses()` with an allowlist.

**📌 Q3. When is `dangerouslySetInnerHTML` acceptable?**
**A.** Rarely—only for **trusted** HTML (e.g., server-generated preview I control). Never for user-supplied content. Prefer editor components (`RichText`, `Notice`) and let the server render final markup.

**📌 Q4. What’s your approach to REST security from blocks?**
**A.** Define `permission_callback` based on capabilities, verify the `X-WP-Nonce` for state-changing requests, define `args` with `type` and `sanitize_callback`, and never leak private data. I also rate-limit or cache server-heavy endpoints.

**📌 Q5. How do you secure URLs, colors, and numbers from block controls?**
**A.** Validate on change in the editor (e.g., URL format, hex color) and sanitize on save (`esc_url_raw`, `sanitize_hex_color`, `absint`). On output: `esc_url`, `esc_attr`, `esc_html`.

**📌 Q6. Why limit block supports and prefer theme tokens?**
**A.** Reducing freeform input (arbitrary HTML/class/styles) shrinks the attack surface and keeps authors on-brand. Theme tokens give safe, curated values that are simple to validate and safe to output.

**📌 Q7. How do you prevent SQL injection in custom features?**
**A.** I never concatenate user input into SQL. I validate/sanitize first, then use `$wpdb->prepare()` for every value. `esc_sql` isn’t a replacement for prepared statements.

**📌 Q8. What about media uploads from blocks?**
**A.** I use the media APIs, verify capabilities, validate MIME using `wp_check_filetype_and_ext()`, and escape URLs with `esc_url` when rendering.

**📌 Q9. One anti-pattern you avoid?**
**A.** I avoid saving raw HTML that users type and then just spitting it back out with `dangerouslySetInnerHTML`.
👉 Instead, I save only structured data (like text, color, size, etc.) in block attributes.
Then, on the server (PHP), I generate the final safe HTML when the block is rendered.
