# ✅ Minimal Required Files

1. **`style.css`**

   - This is **mandatory** because WordPress looks here first to recognize your theme.
   - It must have a **theme header comment** at the very top. Example:

   ```css
   /*
   Theme Name: My First Theme
   Theme URI: https://example.com
   Author: Your Name
   Author URI: https://example.com
   Description: A minimal starter theme.
   Version: 1.0
   License: GNU General Public License v2 or later
   License URI: http://www.gnu.org/licenses/gpl-2.0.html
   Text Domain: my-first-theme
   Tags: custom, theme, minimal
   */
   ```

   🔑 Only **Theme Name** is required, but usually you include others for clarity.

---

2. **`index.php`**

   - Acts as a **fallback template**.
   - WordPress uses this file to render pages if no other specific template (e.g., `single.php`, `page.php`) exists.
   - Example minimal content:

   ```php
   <?php
   get_header();
   ?>

   <h1>Hello, World! This is my first theme.</h1>

   <?php
   get_footer();
   ?>
   ```

---

### ⚡ Optional but Common Early Additions

- **`functions.php`** → for enqueuing CSS/JS, theme setup, etc.
- **`screenshot.png`** → 1200x900 image preview in WP admin.
- **`header.php` + `footer.php`** → to separate markup.

---

### 🗂 Minimal Theme Structure

```
my-first-theme/
│
├── style.css      # Required
├── index.php      # Required
├── functions.php  # Optional, but useful
└── screenshot.png # Optional, admin preview
```

---

👉 So, the absolute minimum = **`style.css` + `index.php`**.
But in practice, you’ll almost always add **`functions.php`** and **`header/footer`** as soon as possible to make the theme usable.
