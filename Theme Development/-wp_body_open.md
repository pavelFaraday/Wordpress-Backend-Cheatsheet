# 🔑 `wp_body_open()` – Interview Cheat Sheet

- **What it is**: A WordPress core **action hook** introduced in **WP 5.2**.
- **Where it goes**: Placed **immediately after `<body>` tag** inside `header.php`.

```php
<body <?php body_class(); ?>>
<?php wp_body_open(); ?>
```

- **Why it exists**: Provides a **standard injection point** for themes/plugins to output content **right after `<body>`**.

- **Use cases**:

  - Accessibility scripts
  - Google Tag Manager / Analytics
  - Tracking pixels
  - Site-wide banners, notices, modals

- **How it works**:

  - Theme calls `wp_body_open()`
  - Plugins (or child themes) hook into it:

    ```php
    add_action('wp_body_open', function() {
      echo '<div class="banner">Welcome!</div>';
    });
    ```

- **Best practice**:

  - Always include it in modern themes (just like `wp_head()` and `wp_footer()`)
  - Ensures **compatibility** with plugins relying on this hook

---

✅ **One-liner for interview**:
"`wp_body_open()` is a core hook added in WP 5.2 that themes must place right after the `<body>` tag so plugins can safely inject code (like tracking scripts or accessibility tools) without editing theme files."
