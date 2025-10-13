# 🧱 Gutenberg Blocks VS Groups VS Patterns (Unsynced, Synced, and Overrides) VS Templates

**Everything in the Gutenberg (Block) Editor is built with blocks.**
From a single paragraph to a full-page layout, everything follows this hierarchy:

> **Blocks → Groups → Patterns → Templates**

Let’s explore each deeply, including **Synced Patterns** and **Overrides**, with features, examples, and use cases.

---

## 🧱 1. **BLOCKS**

### **Definition**

Blocks are the **smallest building units** in the Gutenberg system.
Every individual content piece - heading, paragraph, image, button, or gallery - is a block.

### **Purpose**

To make every piece of content modular, reusable, and independent, allowing editors to build visually without coding.

### **Types**

| Type               | Example                                | Description                                   |
| ------------------ | -------------------------------------- | --------------------------------------------- |
| **Core Blocks**    | `core/paragraph`, `core/image`         | Default blocks shipped with WordPress         |
| **Theme Blocks**   | `core/navigation`, `core/post-title`   | Theme-level layout and data display           |
| **Custom Blocks**  | `myplugin/testimonial`                 | Developer-created blocks with custom behavior |
| **Dynamic Blocks** | `core/query-loop`, `core/latest-posts` | Render via PHP each time page loads           |

### **Key Features**

- Modular and visual editing
- Reusable attributes (color, typography, etc.)
- Full support for style controls via `theme.json`
- Server-side (`render_callback`) or client-side rendering
- React-based editor interface
- Extensible via JS filters (e.g., `blocks.registerBlockType`)

### **Developer Tip**

Use `block.json` for metadata and `register_block_type()` in PHP for consistent block registration between editor and server.

---

## 🧩 2. **GROUPS**

### **Definition**

A **Group** is a **container block** (`core/group`) that holds multiple blocks inside it, managing layout and design collectively.

### **Purpose**

To create sections, align elements, and apply shared settings like background, padding, and layout.

### **Key Features**

- Organizes blocks visually and semantically
- Shared styling: background, color, borders, spacing
- Layout options: **Flow**, **Flex**, or **Grid**
- Supports responsive design and semantic tags (`div`, `section`, `header`, etc.)
- Can be nested (groups inside groups)
- Can be locked for structural control (`templateLock="all"`)

### **Example**

Use a Group block for a **Hero Section** that includes a heading, paragraph, and button, all sharing the same background color and padding.

---

## 🖼 3. **PATTERNS**

### **Definition**

Patterns are **pre-designed block layouts** that users can insert with one click.
They can represent sections (e.g., hero banners, pricing tables, testimonials) or entire page templates.

### **Purpose**

To speed up page creation with brand-consistent, ready-to-use layouts.

### **Types of Patterns**

| Type                                | Description                                                                          | Example                                                 |
| ----------------------------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------------- |
| **Unsynced (Regular)**              | One-time insertion - once placed, becomes independent blocks.                        | Hero section pattern used and customized per page       |
| **Synced (Reusable)**               | Global pattern shared across pages - edit once, updates everywhere.                  | A global "Contact Banner" used on every page            |
| **Synced with Overrides (WP 6.5+)** | Layout stays linked globally, but some inner blocks can be edited uniquely per page. | Shared “Hero Layout” with unique headlines on each page |

### **Main Features**

- Predefined block arrangements with styling and structure
- Insertable via **Patterns tab** or **Pattern Explorer**
- Editable after insertion (if unsynced)
- Grouped by categories for quick access
- Can include locked areas to prevent layout changes
- Registered via PHP (`register_block_pattern()`) or JSON (`patterns/*.json`)
- Support **pattern overrides** for flexible synced content
- Can be **distributed via theme**, **plugin**, or **WordPress.org Pattern Directory**

### **Developer Example (PHP)**

```php
register_block_pattern(
  'mytheme/hero-section',
  [
    'title' => __( 'Hero Section', 'mytheme' ),
    'content' => '<!-- wp:group -->
      <!-- wp:heading {"level":1} -->Hero Title<!-- /wp:heading -->
      <!-- wp:paragraph -->Your subtitle goes here.<!-- /wp:paragraph -->
      <!-- wp:button {"text":"Learn More"} /-->
    <!-- /wp:group -->',
  ]
);
```

### **JSON Example**

```json
{
  "title": "Hero Section",
  "slug": "mytheme/hero-section",
  "categories": ["hero", "cta"],
  "content": "<!-- wp:group --><!-- wp:heading -->Hero Title<!-- /wp:heading --><!-- /wp:group -->"
}
```

### 🧩 **Synced Patterns**

**Definition:**
Synced Patterns (formerly _Reusable Blocks_) are **global instances** of a block group.
==Editing the content or design in one instance automatically updates all others across the site.==

**Purpose:**
To maintain consistency for shared elements (like headers, callouts, or footers).

**Features**

- Saved in the database and accessible across posts/pages.
- Editable in one place, reflected everywhere.
- Can be locked to protect layout.
- Best for global content (e.g., “Call-to-Action banner” repeated on every page).
- Registered through the editor UI (right-click → _Create Pattern → Synced_).

**Example Use Case**

> Add a "Join Our Newsletter" section to 10 pages. Update text or design once — all pages reflect it automatically.

### 🧩 **Synced Pattern Overrides (New in WP 6.5+)**

**Definition:**
Overrides let **specific inner blocks** within a Synced Pattern be **editable per instance**, while the rest stay globally synced.

**Purpose:**
To mix **global design consistency** with **localized content flexibility**.

**How It Works:**

- The pattern’s layout and styling remain synced.
- Inner blocks marked as _overridable_ can have unique content in each location.
- Overrides are defined in the pattern’s markup using the `overrides` property.

**Example**

```html
<!-- wp:pattern {
  "slug": "mytheme/hero-pattern",
  "overrides": {
    "core/heading": ["content"],
    "core/paragraph": ["content"]
  }
} /-->
```

In this example:

- The design (colors, spacing, structure) stays global.
- Editors can change only the heading and paragraph text per instance.

**Use Case**

> A company uses one global “Hero Layout” pattern site-wide. Each page can customize the headline and description, but not the layout or button style.

**Benefits**

- Keeps brand identity consistent.
- Enables page-specific personalization.
- Simplifies maintenance (design updates apply everywhere).

### **Summary of Pattern Types**

| Pattern Type                    | Editable Locally? | Updates Globally?              | Example Use                             |
| ------------------------------- | ----------------- | ------------------------------ | --------------------------------------- |
| **Unsynced Pattern**            | Yes               | No                             | Page-specific hero or section           |
| **Synced Pattern**              | No (global only)  | Yes                            | Shared CTA banner                       |
| **Synced Pattern w/ Overrides** | Partially         | Yes (except overridden fields) | Shared design with unique text per page |

---

## 🧭 4. **TEMPLATES**

### **Definition**

Templates are **full-page or post layouts** made from blocks and template parts.
They define how WordPress content (posts, pages, products, etc.) is displayed.

### **Purpose**

To enforce consistent structure and design across the entire site or content type.

### **Main Features**

| Feature                      | Description                                                             |
| ---------------------------- | ----------------------------------------------------------------------- |
| **Full-page layout**         | Controls header, content, and footer for specific contexts.             |
| **Stored in block themes**   | `/templates/*.html` or `/parts/*.html`.                                 |
| **Dynamic content**          | Uses data-driven blocks (`Post Title`, `Content`, `Query Loop`).        |
| **Editable in Site Editor**  | Can be customized visually.                                             |
| **Template hierarchy**       | Follows WP hierarchy (e.g., `single.html`, `archive.html`, `404.html`). |
| **Template parts**           | Reusable blocks like header and footer.                                 |
| **Locking support**          | Protects structure with `template_lock`.                                |
| **Custom post type support** | Define default block trees in PHP using `register_post_type()`.         |

### **Developer Example**

```php
register_post_type( 'team', [
  'show_in_rest'  => true,
  'template'      => [
    [ 'core/heading', [ 'content' => 'Team Name' ] ],
    [ 'core/paragraph', [ 'content' => 'Short bio...' ] ],
    [ 'core/image' ],
  ],
  'template_lock' => 'insert',
] );
```

### **Use Cases**

- Define the layout for single posts, pages, or product pages.
- Standardize design across the site.
- Use dynamic blocks for post data.
- Lock structure for non-technical editors.

---

## ⚙️ COMPARISON TABLE

| Feature               | **Blocks**             | **Groups**       | **Patterns (Unsynced)** | **Synced Patterns**     | **Synced w/ Overrides**          | **Templates**          |
| --------------------- | ---------------------- | ---------------- | ----------------------- | ----------------------- | -------------------------------- | ---------------------- |
| **Scope**             | Single content unit    | Layout container | Section layout          | Global reusable section | Shared layout + unique content   | Full-page layout       |
| **Editable locally**  | Yes                    | Yes              | Yes                     | No                      | Partially (overrides only)       | Yes (if not locked)    |
| **Global updates**    | No                     | No               | No                      | Yes                     | Yes (except overrides)           | Yes (in Site Editor)   |
| **Where defined**     | block.json / PHP       | Core block       | PHP / JSON              | Editor UI               | Markup + overrides JSON          | Theme templates folder |
| **Purpose**           | Smallest content piece | Organize layout  | Prebuilt sections       | Reusable global layout  | Global layout with local content | Full-page structure    |
| **Dynamic data**      | Optional               | No               | Optional                | Optional                | Optional                         | Yes                    |
| **Locking available** | Yes                    | Yes              | Yes                     | Yes                     | Yes                              | Yes                    |
| **UI Location**       | Inserter               | Editor           | Patterns tab            | Patterns library        | Patterns library                 | Site Editor            |

---

## 🔁 RELATIONSHIP FLOW

```
Blocks → Groups → Patterns → Templates
```

- **Blocks**: Basic content units.
- **Groups**: Structure and style multiple blocks.
- **Patterns**: Prebuilt layouts of grouped blocks.
- **Synced Patterns**: Global reusable layouts.
- **Overrides**: Flexible synced layouts with local content edits.
- **Templates**: Full-page blueprints combining all of the above.

---

## 💡 In Summary

| Concept             | Core Idea                                          |
| ------------------- | -------------------------------------------------- |
| **Blocks**          | Smallest content units - everything starts here.   |
| **Groups**          | Organize and style multiple blocks together.       |
| **Patterns**        | Prebuilt reusable sections (local or global).      |
| **Synced Patterns** | Global reusable sections that update everywhere.   |
| **Overrides**       | Synced layouts with per-page editable fields.      |
| **Templates**       | Full-page layouts that define site-wide structure. |

---

👉 **In short:**

- **Blocks** build content.
- **Groups** organize layout.
- **Patterns** provide reusable designs.
- **Synced Patterns** maintain global layouts.
- **Overrides** personalize synced layouts.
- **Templates** control the overall page or site structure.
