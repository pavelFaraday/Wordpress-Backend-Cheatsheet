# 🧩 PHP Filters in Gutenberg

### 💡 In one line:

> PHP filters let you control **which blocks appear**, **how they behave**, and **how they render** – all from the **server side (PHP)**, before or during the editor and front-end output.

## 🧠 Why They Exist (Purpose)

Gutenberg blocks are mostly registered in **JavaScript**, but:

- Some policies need **server logic** (roles, post types, templates).
- Some blocks need **standardized styles/supports** site-wide.
- Some outputs need **conditional rendering**.

That’s where **PHP filters (server-side hooks)** come in.

They allow you to:

1. **Curate** what blocks users can insert.
2. **Modify** block registration (supports, styles, attributes).
3. **Control** how blocks render (add classes, hide, replace).
4. **Organize** blocks inside the editor categories.

---

## 🏗️ 1. Control What Blocks Are Available

### **Hook:** `allowed_block_types_all`

**Purpose:**
Define which blocks can be used in the editor (by post type, user role, or template).

**Core Concept:**
Return either:

- `true` → allow all blocks
- `false` → block all
- `['block/name', ...]` → allow only specific blocks

**Example:**

```php
add_filter('allowed_block_types_all', function ($allowed, $context) {
    if (isset($context->post) && $context->post->post_type === 'product') {
        return ['core/heading', 'core/paragraph', 'core/image', 'woocommerce/handpicked-products'];
    }
    return true;
}, 10, 2);
```

**Use Cases:**

- Limit blocks on product or landing pages.
- Give editors full access but restrict authors.
- Maintain brand consistency and avoid messy layouts.

---

## ⚙️ 2. Change Block Settings During Registration

### A) `block_type_metadata`

**Purpose:**
Modify the **raw block.json data** _before_ WordPress processes it.

**Example:**

```php
add_filter('block_type_metadata', function ($metadata) {
    $metadata['apiVersion'] = 2;
    return $metadata;
});
```

**When to Use:**
When you want to adjust a base property for every block globally.

---

### B) `block_type_metadata_settings`

**Purpose:**
Change the **derived settings** (supports, styles, variations) for all blocks after WP reads block.json.

**Example:**

```php
add_filter('block_type_metadata_settings', function ($settings, $metadata) {
    $supports = $settings['supports'] ?? [];
    $supports['spacing'] = ['margin' => true, 'padding' => true];
    $supports['border']  = true;
    $settings['supports'] = $supports;
    return $settings;
}, 10, 2);
```

**Use Cases:**

- Enable margin, padding, border for all blocks.
- Add global styles (like `rounded` or `shadow`).
- Apply common supports site-wide without editing block code.

---

### C) `register_block_type_args`

**Purpose:**
Modify final registration arguments for **a specific block**.

**Example:**

```php
add_filter('register_block_type_args', function ($args, $name) {
    if ($name === 'core/image') {
        $args['supports']['align'] = ['wide', 'full'];
        $args['styles'][] = ['name' => 'rounded', 'label' => 'Rounded'];
    }
    return $args;
}, 10, 2);
```

**Use Cases:**

- Add style variation to one block.
- Enable full-width alignment for specific blocks.
- Tweak block settings without touching JS code.

---

## 🧱 3. Modify or Replace HTML Output

These filters run during block **rendering** (when HTML is being generated).

### A) `render_block`

**Purpose:**
Modify the HTML of **every** block on the front end.

**Example:**

```php
add_filter('render_block', function ($content, $block) {
    if ($block['blockName'] === 'core/paragraph') {
        $content = str_replace('<p', '<p class="is-highlighted"', $content);
    }
    return $content;
}, 10, 2);
```

**Use Cases:**

- Add CSS classes globally.
- Wrap content with custom HTML.
- Small branding tweaks.

⚠️ Keep it light! Runs for _every block_.

### B) `render_block_{name}`

**Purpose:**
Modify output only for one block type (e.g., `core/image`).

**Example:**

```php
add_filter('render_block_core/image', function ($content) {
    return str_replace('<figure', '<figure class="has-shadow"', $content);
}, 10, 1);
```

**Use Case:**
When you want to change output only for one specific block efficiently.

### C) `pre_render_block`

**Purpose:**
Skip or replace a block’s rendering **before** it happens.

**Example:**

```php
add_filter('pre_render_block', function ($pre, $parsed) {
    if ($parsed['blockName'] === 'core/quote' && is_search()) {
        return ''; // hide quotes on search pages
    }
    return $pre;
}, 10, 2);
```

**Use Cases:**

- Hide blocks in certain templates or pages.
- Replace output conditionally.

### D) `render_block_data`

**Purpose:**
Change attributes _before_ rendering happens.

**Example:**

```php
add_filter('render_block_data', function ($block) {
    if ($block['blockName'] === 'core/image') {
        $block['attrs']['className'] .= ' has-border';
    }
    return $block;
});
```

**Use Case:**

- Add or modify block attributes dynamically.

---

## 🧭 4. Organize Inserter Categories

**Hook:** `block_categories_all`

**Purpose:**
Add or reorder block categories in the Gutenberg inserter panel.

**Example:**

```php
add_filter('block_categories_all', function ($categories) {
    $categories[] = [
        'slug'  => 'custom-blocks',
        'title' => __('Custom Blocks', 'textdomain'),
    ];
    return $categories;
});
```

**Use Case:**
Make your plugin or theme blocks easier to find in the inserter.

---

## ⚡ Common Real-World Scenarios

| Scenario                                | Hook                                   | Why                     |
| --------------------------------------- | -------------------------------------- | ----------------------- |
| Allow specific blocks for landing pages | `allowed_block_types_all`              | Clean author experience |
| Enable spacing and border globally      | `block_type_metadata_settings`         | Standardized styling    |
| Add custom style to one block           | `register_block_type_args`             | Avoid editing core code |
| Add custom wrapper or class             | `render_block` / `render_block_{name}` | Small HTML tweaks       |
| Hide block in certain views             | `pre_render_block`                     | Conditional rendering   |
| Group custom blocks                     | `block_categories_all`                 | Better organization     |

---

## 🧩 Core Concepts to Remember

- **“Registration” vs “Rendering”:**
  Registration = how block behaves in the editor.
  Rendering = how it appears on the front end.

- **Prefer theme.json** for static settings (color, spacing, etc).
  Use PHP filters only when logic or dynamic control is needed.

- **Choose the earliest hook possible:**
  Registration-time hooks are safer and faster than render-time edits.

- **Guard your conditions:**
  Always check block name or context (post type, user role).

- **Escape and validate:**
  Treat block attributes like user input when adding custom HTML.

---

## 🧾 Quick Cheat-Sheet

| Goal                 | Hook                           | Runs When             | Return               | Example Use                 |
| -------------------- | ------------------------------ | --------------------- | -------------------- | --------------------------- |
| Limit blocks         | `allowed_block_types_all`      | Editor loads          | true / false / array | Allow only certain blocks   |
| Edit raw metadata    | `block_type_metadata`          | During registration   | array                | Adjust apiVersion           |
| Bulk adjust settings | `block_type_metadata_settings` | After metadata        | array                | Enable spacing globally     |
| Tweak per-block      | `register_block_type_args`     | Final registration    | array                | Add styles to core/image    |
| Change block HTML    | `render_block`                 | On every render       | string               | Add class to <p>            |
| Target one block     | `render_block_{name}`          | Specific block render | string               | Wrap quote block            |
| Skip output          | `pre_render_block`             | Before render         | string/null          | Hide block                  |
| Modify attrs         | `render_block_data`            | Before render         | array                | Add custom class            |
| Add category         | `block_categories_all`         | Inserter loads        | array                | Add “Custom Blocks” section |

---

## PHP Hooks VS Filter Hooks

### ✅ PHP Filter = Filter Hook

In WordPress, a **PHP filter** **is a type of hook** — specifically, a **“filter hook.”**

## 🧩 WordPress has two kinds of hooks:

| Type            | Purpose                                                                                  | Example                                              |
| --------------- | ---------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| **Action Hook** | Runs a custom function at a specific point — it performs an action, doesn’t return data. | `add_action( 'init', 'my_function' );`               |
| **Filter Hook** | Passes data through your function — it expects you to return something (often modified). | `add_filter( 'the_content', 'my_filter_function' );` |

So in short:

> A **filter hook** takes data → runs it through your function → and expects you to return it back (changed or unchanged).

## ✅ So in summary:

| Concept                      | Definition                                                                  |
| ---------------------------- | --------------------------------------------------------------------------- |
| **Hook**                     | Generic system to extend or change WordPress behavior.                      |
| **Action Hook**              | Runs your code **without returning anything**.                              |
| **Filter Hook (PHP Filter)** | Runs your code **and expects modified data back**.                          |
| **PHP Filters in Gutenberg** | Filter hooks specifically affecting blocks (allowed, registered, rendered). |

**In short:**

> All the “PHP filters” in Gutenberg (like `allowed_block_types_all`, `render_block`, etc.) are **filter hooks** — meaning, they intercept data, let you modify it, and return it back to WordPress.

---

## 💬 Summary (Perfect for Interview)

> “PHP filters in Gutenberg let me manage the editor and rendering from the server side.
> I can decide which blocks are allowed (`allowed_block_types_all`),
> modify block settings during registration (`block_type_metadata_settings`, `register_block_type_args`),
> and control front-end output (`render_block`, `pre_render_block`).
> It’s powerful for enforcing design systems, simplifying author experience, and applying logic based on user role, post type, or context.”
