# `get_header()` & `get_footer()`

## 🧩 What is `get_header()`?

- It’s a **WordPress function** that loads the file **`header.php`** from your theme.
- Usually placed **at the top of your template files** (like `index.php`, `single.php`, `page.php`).
- It’s used to keep your theme **organized** and avoid repeating the same HTML/PHP code everywhere.

👉 Example:

```php
<?php get_header(); ?>

<h1>Welcome to my site</h1>

<?php get_footer(); ?>
```

---

### 📦 What does `header.php` contain?

Typically, it contains:

- The **doctype** and opening `<html>` + `<head>` tags
- The site’s `<title>` (added by WordPress automatically if you use `wp_head()`)
- Stylesheets and scripts
- The opening `<body>` tag
- The site header (logo, navigation menu, etc.)

👉 Example `header.php`:

```php
<!DOCTYPE html>
<html <?php language_attributes(); ?>>
<head>
  <meta charset="<?php bloginfo('charset'); ?>">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <?php wp_head(); ?>
</head>
<body <?php body_class(); ?>>

<header>
  <h1><?php bloginfo('name'); ?></h1>
  <p><?php bloginfo('description'); ?></p>
  <?php wp_nav_menu(); ?>
</header>
```

⚡ Notice `wp_head()` → this is **very important** because WordPress (and plugins) hook into it to load scripts, styles, SEO tags, etc.

---

## 🧩 What is `get_footer()`?

- It loads the file **`footer.php`** from your theme.
- Usually placed **at the bottom of your template files**.

👉 Example:

```php
<?php get_footer(); ?>
```

---

### 📦 What does `footer.php` contain?

Typically, it contains:

- The closing `<footer>` HTML with copyright or widgets
- The `wp_footer()` hook (needed for scripts and plugins)
- Closing `</body>` and `</html>` tags

👉 Example `footer.php`:

```php
<footer>
  <p>&copy; <?php echo date('Y'); ?> My WordPress Site</p>
</footer>

<?php wp_footer(); ?>
</body>
</html>
```

⚡ Notice `wp_footer()` → this is **just as important** as `wp_head()` because many scripts (like analytics, plugins, and theme JS) are injected here.

---

## 🧑‍🏫 Simple Summary

- **`get_header()`** → pulls in **header.php** (opening part of page).
- **`get_footer()`** → pulls in **footer.php** (closing part of page).
- They keep your theme **modular**, so you don’t repeat HTML everywhere.

---

## ⚡ Bonus: Advanced Usage

Both functions accept a **parameter** if you want different headers/footers.

```php
<?php get_header('custom'); ?>
```

This looks for **`header-custom.php`** in your theme.
👉 Useful if you want a different header for blog posts vs landing pages.

---

✅ Without these two functions (and their hooks `wp_head()` and `wp_footer()`), many WordPress features and plugins won’t work properly.
