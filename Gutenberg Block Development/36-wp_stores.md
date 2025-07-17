# 🧠 What is `wp.data` in WordPress?

`wp.data` is a **JavaScript module** in WordPress that provides a **centralized way to manage state** (data) in the WordPress admin, especially in the **Gutenberg (Block) Editor**. It's part of the WordPress JavaScript API and is built on top of the **Redux pattern**.

Think of `wp.data` as a smart assistant that:

* 📖 **Reads data** (`select`)
* ✏️ **Updates data** (`dispatch`)
* 👀 **Watches data changes** (`subscribe`)
* 🧩 **Registers custom data stores**

It makes interacting with WordPress data (like posts, users, blocks, etc.) much easier and more consistent in JavaScript.

---

## 💡 Why `wp.data` Exists

Before `wp.data`, WordPress JavaScript code was messy and inconsistent. Developers had to:

* Write custom AJAX calls
* Manage state in individual components
* Repeat logic across features

`wp.data` fixes that by:
✅ Centralizing data
✅ Avoiding prop drilling
✅ Making components reactive
✅ Providing a shared data layer for all plugins and core features

---

## 🏗️ Core Concepts

| Concept           | Description                | Example                                               |
| ----------------- | -------------------------- | ----------------------------------------------------- |
| `select()`        | Read data from a store     | `select('core').getEntityRecords('postType', 'post')` |
| `dispatch()`      | Update or trigger actions  | `dispatch('core/editor').editPost({...})`             |
| `subscribe()`     | Watch for state changes    | `subscribe(() => console.log('Something changed'))`   |
| `registerStore()` | Create your own data store | Used in custom plugin development                     |

---

## 🛒 Real Example Use Cases

### 1. Get current post title

```js
const title = wp.data.select('core/editor').getEditedPostAttribute('title');
```

### 2. Change the post content

```js
wp.data.dispatch('core/editor').editPost({ content: '<p>Hello world</p>' });
```

### 3. Show a success notice

```js
wp.data.dispatch('core/notices').createSuccessNotice('Post saved!');
```

### 4. Insert a new paragraph block

```js
const { createBlock } = wp.blocks;
wp.data.dispatch('core/block-editor').insertBlock(createBlock('core/paragraph'));
```

---

## 🔍 Which Stores Are Inside `wp.data`?

`wp.data` holds many **"stores"**, which are like mini databases for specific WordPress features. Examples:

* `core`: WordPress data (posts, users, etc.)
* `core/editor`: The current post
* `core/block-editor`: Block data
* `core/notices`: Admin notices
* `core/preferences`: User settings
* …and more

You can list them with:

```js
console.log(wp.data.getStoreNames());
```

---

## 🔄 Watch for Data Changes (Reactively)

```js
const unsubscribe = wp.data.subscribe(() => {
    const title = wp.data.select('core/editor').getEditedPostAttribute('title');
    console.log('New title:', title);
});
```

---

## ✅ Summary

| Feature                 | What it Does               |
| ----------------------- | -------------------------- |
| `wp.data.select`        | Reads state (get data)     |
| `wp.data.dispatch`      | Writes/updates state       |
| `wp.data.subscribe`     | Reacts to state changes    |
| `wp.data.registerStore` | Create your own data store |

---

> 🧠 In short: `wp.data` gives developers superpowers to **manage and react to data** in WordPress—cleanly, consistently, and without needing to mess with AJAX or manual DOM updates.

# 🧠 `wp.data` Stores in WordPress

The `wp.data` module is a **state management library** used across WordPress, especially in the Gutenberg block editor and admin UI. It is built on principles similar to **Redux**, allowing developers to **retrieve**, **subscribe to**, and **dispatch changes** to shared data stores in a predictable way.

It provides a centralized way to:

- Access data (`select`)
- Modify data (`dispatch`)
- Watch for data changes (`subscribe`)
- Register your own stores (`registerStore`)

---

# 🔍 Core Concepts of `wp.data`

| Concept         | Description                  |
| --------------- | ---------------------------- |
| `select`        | Read state from a store      |
| `dispatch`      | Write/update state           |
| `subscribe`     | Listen to changes in state   |
| `registerStore` | Register a custom data store |

---

# 📦 List of All **Core Stores** in `wp.data` with Purpose & Examples

Here’s a **comprehensive list** of the most commonly used core stores in `wp.data`.

---

## 1. `core`

### 📌 Purpose:

Handles general WordPress data like settings, users, taxonomies, posts, and more.

### ✅ Example Use Cases:

```js
// Get all categories
const categories = wp.data
  .select("core")
  .getEntityRecords("taxonomy", "category");

// Get current user data
const currentUser = wp.data.select("core").getCurrentUser();
```

---

## 2. `core/editor`

### 📌 Purpose:

Manages the current post being edited (pre-Gutenberg post editor API).

### ✅ Example Use Cases:

```js
// Get the current post title
const title = wp.data.select("core/editor").getEditedPostAttribute("title");

// Update the post content
wp.data.dispatch("core/editor").editPost({ content: "<p>Hello world!</p>" });
```

---

## 3. `core/block-editor`

### 📌 Purpose:

Handles data related to blocks in the block editor.

### ✅ Example Use Cases:

```js
// Get all blocks
const blocks = wp.data.select("core/block-editor").getBlocks();

// Insert a paragraph block
wp.data
  .dispatch("core/block-editor")
  .insertBlock(wp.blocks.createBlock("core/paragraph"));
```

---

## 4. `core/edit-post`

### 📌 Purpose:

Controls the post editor UI (panels, sidebar, etc.).

### ✅ Example Use Cases:

```js
// Check if the block inserter is open
const isInserterOpen = wp.data.select("core/edit-post").isInserterOpened();

// Toggle the sidebar
wp.data.dispatch("core/edit-post").toggleGeneralSidebar("edit-post/block");
```

---

## 5. `core/edit-site`

### 📌 Purpose:

Used in Full Site Editing (FSE). Manages data for templates, template parts, and site-wide editing.

### ✅ Example Use Cases:

```js
// Get current template ID
const templateId = wp.data
  .select("core/edit-site")
  .getEditedEntityRecord("postType", "wp_template");
```

---

## 6. `core/notices`

### 📌 Purpose:

Handles application notices (success, error, info messages).

### ✅ Example Use Cases:

```js
// Create a success notice
wp.data.dispatch("core/notices").createSuccessNotice("Saved successfully!");

// Remove notice
wp.data.dispatch("core/notices").removeNotice("your-notice-id");
```

---

## 7. `core/interface`

### 📌 Purpose:

Manages generic UI state (active panels, modals, preferences, etc.)

### ✅ Example Use Cases:

```js
// Open a generic sidebar
wp.data.dispatch("core/interface").openGeneralSidebar("my-plugin/sidebar");

// Check if the sidebar is open
const isOpen = wp.data
  .select("core/interface")
  .isSidebarOpened("my-plugin/sidebar");
```

---

## 8. `core/viewport`

### 📌 Purpose:

Tracks the current viewport (desktop, tablet, mobile) – useful in responsive design in the editor.

### ✅ Example Use Cases:

```js
const deviceType = wp.data.select("core/viewport").getDeviceType(); // 'Desktop'
```

---

## 9. `core/keyboard-shortcuts`

### 📌 Purpose:

Handles registered keyboard shortcuts.

### ✅ Example Use Cases:

```js
// Register a custom shortcut
wp.data.dispatch("core/keyboard-shortcuts").registerShortcut("myplugin.save", {
  category: "global",
  description: "Save custom data",
  keyCombination: { modifier: "primary", character: "s" },
});
```

---

## 10. `core/preferences`

### 📌 Purpose:

Stores user-specific preferences (like collapsed panels, etc.).

### ✅ Example Use Cases:

```js
// Set a user preference
wp.data.dispatch("core/preferences").set("my-plugin", "showSidebar", true);

// Read the preference
const showSidebar = wp.data
  .select("core/preferences")
  .get("my-plugin", "showSidebar");
```

---

## 11. `core/widgets`

### 📌 Purpose:

Manages legacy widgets and widget areas (important for block-based widget editor).

### ✅ Example Use Cases:

```js
const widgetAreas = wp.data.select("core/widgets").getWidgetAreas();
```

---

## 12. `core/reusable-blocks`

### 📌 Purpose:

Manage reusable blocks (saved blocks across posts/pages).

### ✅ Example Use Cases:

```js
const reusableBlocks = wp.data
  .select("core/reusable-blocks")
  .getReusableBlocks();
```

---

## 13. `core/nux` (New User Experience)

### 📌 Purpose:

Shows tips and guidance for first-time users in the editor.

### ✅ Example Use Cases:

```js
wp.data.dispatch("core/nux").triggerGuide("my-plugin/intro-tip");
```

---

## 14. `core/plugins`

### 📌 Purpose:

Handles admin screen plugin panel registration.

### ✅ Example Use Cases:

```js
wp.data.dispatch("core/plugins").registerPlugin("my-plugin", {
  render: MyPluginSidebar,
  icon: "admin-plugins",
});
```

---

## 15. `core/customize`

### 📌 Purpose:

Used in the Customizer to manage settings and panels.

### ✅ Example Use Cases:

> Mostly used internally within the Customizer, not common in block development.

---

## 16. `core/block-directory`

### 📌 Purpose:

Fetches blocks from the WordPress block directory (when users search in inserter).

### ✅ Example Use Cases:

```js
const results = wp.data.select("core/block-directory").getResults("carousel");
```

---

## 17. `core/edit-navigation`

### 📌 Purpose:

Used in the Navigation block editor (menus, navigation structure).

---

## 18. `core/block-patterns`

### 📌 Purpose:

Returns registered block patterns.

### ✅ Example Use Cases:

```js
const patterns = wp.data.select("core/block-patterns").getBlockPatterns();
```

---

# 🧪 Advanced: Check All Registered Stores Dynamically

```js
console.log(wp.data.getStoreNames());
```

# 🔄 Subscribe to Store Changes

```js
const unsubscribe = wp.data.subscribe(() => {
  const title = wp.data.select("core/editor").getEditedPostAttribute("title");
  console.log("Title updated:", title);
});
```

You can call `unsubscribe()` to stop listening.

---

# 💡 Summary

| Task                            | Store               |
| ------------------------------- | ------------------- |
| Get posts, taxonomies, settings | `core`              |
| Edit post attributes            | `core/editor`       |
| Manage blocks in editor         | `core/block-editor` |
| Manage editor UI                | `core/edit-post`    |
| Manage notices                  | `core/notices`      |
| User preferences                | `core/preferences`  |
| Responsive preview state        | `core/viewport`     |
| Site editing (FSE)              | `core/edit-site`    |

---

