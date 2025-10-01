# SlotFill API (Plugin UI Extensions) in Gutenberg

## What is SlotFill (in plain English)?

- **Slot** = an empty “hole” in the Gutenberg editor UI (like a reserved spot).
- **Fill** = your plugin’s UI content that gets inserted into that hole.
  👉 Together, Slot + Fill = **SlotFill**.

WordPress core defines where the holes (slots) are — sidebars, panels, menus, toolbars, etc.
You (plugin dev) decide what to fill them with.

## 🧩 Why SlotFill Exists

- Before Gutenberg, extending the editor UI was messy (PHP hooks, jQuery hacks).
- Now with SlotFill, it’s **clean and React-based**.
- It ensures **multiple plugins can inject UI safely** without breaking each other.
- Works the same across both **Post Editor** and **Site Editor** (since WP 6.6).

## 📍 The Most Common Slots You Can Use

Here are the main “injection points” you get:

1. **PluginSidebar** → Your own sidebar, opened from a toolbar button or the ⋯ menu.
   _(Great for big workflows like SEO tools, editorial checklists, asset libraries.)_

2. **PluginDocumentSettingPanel** → A panel inside the normal “Document” sidebar.
   _(Perfect for things tied to the post/page itself — SEO title, canonical URL, etc.)_

3. **PluginSidebarMoreMenuItem** → An item in the ⋯ “More” menu that toggles your sidebar.
   _(Auto-created when you use PluginSidebar, but you can customize it.)_

4. **PluginPostStatusInfo** → Add info under the post status area.
   _(e.g., “Word count: 523” or “SEO score: Good.”)_

5. **PluginPrePublishPanel** → A panel that shows up **before publishing**.
   _(Useful for final checks like “Alt text missing” or “Word count too low.”)_

6. **PluginPostPublishPanel** → A panel that shows up **after publishing**.
   _(Great for “Next steps” like share buttons, checklists, etc.)_

7. **PluginBlockSettingsMenuItem** → Extra items in the ⋯ menu of a **selected block**.
   _(e.g., “Convert this block to custom format.”)_

8. **PluginMoreMenuItem** → Extra item in the **top toolbar’s ⋯ menu**.
   _(Good for utilities or links, like “Open Help Center.”)_

9. **MainDashboardButton** → A button in the top toolbar itself.
   _(For quick global actions, like toggling a custom panel.)_

## 🔄 How It Works in Practice

1. WordPress says: “Here’s a slot in the editor UI.”
2. You say: “I’ll fill it with my controls.”
3. Gutenberg takes care of placing it in the right place, with the right styles.
4. When multiple plugins add fills to the same slot → no problem, they just stack.

## ⚖️ When to Use What

- **Big workflows?** → Use a **PluginSidebar**.
- **Post/page settings?** → Use a **PluginDocumentSettingPanel**.
- **Warnings before publishing?** → Use a **PluginPrePublishPanel**.
- **Tips after publishing?** → Use a **PluginPostPublishPanel**.
- **Extra block actions?** → Use a **PluginBlockSettingsMenuItem**.
- **Quick toolbar items?** → Use **PluginMoreMenuItem** or **MainDashboardButton**.

## 🚦 Common Pitfalls

- Using **old APIs** (like `wp.editPost.*`) on new WordPress (≥6.6) → causes deprecation warnings.
- Forgetting to **register post meta with REST support** → your data won’t save.
- Using the **same panel name as another plugin** → it won’t toggle properly.
- Missing **script dependencies** → nothing shows up.

## 📝 The Interview Elevator Pitch

“If I need to add custom UI into Gutenberg, I use the **SlotFill API**. Core defines the **slots** (like sidebars, panels, menus), and I create **fills** to inject my plugin’s UI there. The most common ones are **PluginSidebar** for a whole new sidebar and **PluginDocumentSettingPanel** for fields inside the document sidebar. There are also slots for pre-publish, post-publish, block menus, and more. Since WP 6.6, everything is unified under `@wordpress/editor`, so I avoid deprecated `wp.editPost.*` APIs. SlotFill gives me a clean, React-based way to extend the editor without conflicts.”

[SlotFill API - PowerPoint](<JS Libraries/SlotFill_API_Gutenberg_Plugin_UI_Extensions.pptx>)

**SlotFill** is a React portal-style extensibility pattern used by the WordPress Block Editor. Core exposes **Slots** (fixed “mount points” in the editor UI), and plugins render **Fills** to those Slots. Multiple Fills can target the same Slot; the editor handles layout/ordering.

Gutenberg ships a set of **predefined SlotFills** for common areas: sidebar panels, the More (⋯) menu, pre-publish checks, block settings menu items, and more. You register your UI by calling **`registerPlugin`** (from `@wordpress/plugins`) and rendering the provided components (e.g., `<PluginSidebar/>`, `<PluginDocumentSettingPanel/>`).


---

## Core concepts & moving parts

- **Slots & Fills**: Think “placeholders” (Slots) that accept “content injections” (Fills). Rendering is virtual/portaled so your UI appears in the intended editor location.
- **`registerPlugin( name, { render, icon, scope } )`**: Attaches your React app to the editor and makes your SlotFills live. Your `render` returns any number of SlotFills.
- **Predefined SlotFills** (selected highlights):

  - `PluginSidebar` + `PluginSidebarMoreMenuItem` — togglable custom sidebar in post/site editor toolbars and the ⋯ menu. 
  - `PluginDocumentSettingPanel` — a **panel inside the built-in “Post/Page” settings sidebar** (document tab). Namespaced by your plugin. 
  - Others you’ll meet in practice: `PluginPostStatusInfo`, `PluginPrePublishPanel`, `PluginPostPublishPanel`, `PluginBlockSettingsMenuItem`

---

## Minimal, modern examples (ready to paste)

### 1) PHP bootstrap: enqueue editor script with correct deps

```php
<?php
/**
 * Plugin Name: My Editorial Sidebar
 */
add_action( 'enqueue_block_editor_assets', function () {
	wp_enqueue_script(
		'my-editorial-sidebar',
		plugins_url( 'build/editor-sidebar.js', __FILE__ ),
		// Key deps for SlotFill/Sidebar in WP 6.6+:
		[ 'wp-element', 'wp-components', 'wp-editor', 'wp-plugins', 'wp-data', 'wp-core-data', 'wp-compose', 'wp-i18n' ],
		null,
		true
	);
} );
```

> Why these dependencies? You’re using React (`wp-element`), UI components (`wp-components`), unified SlotFills (`wp-editor`), plugin registration (`wp-plugins`), and data stores (`wp-data`, `wp-core-data`). The `wp-plugins` dependency in particular is required for `registerPlugin`. ([WordPress Developer Resources][6], [Stack Overflow][9])

---

### 2) A custom **PluginSidebar** (your own sidebar with controls)

```js
// build/editor-sidebar.js
const { registerPlugin } = wp.plugins;
const { PluginSidebar, PluginSidebarMoreMenuItem } = wp.editor; // WP 6.6+ unified
const { PanelBody, TextControl, ToggleControl } = wp.components;
const { useSelect, useDispatch } = wp.data;
const { store: coreEditorStore } = wp.editor; // for editor actions/selectors
const { __ } = wp.i18n;

function EditorialSidebar() {
  // Example: read/write a custom post meta field
  const meta = useSelect(
    (select) => select("core/editor").getEditedPostAttribute("meta") || {},
    []
  );
  const { editPost } = useDispatch("core/editor");

  const value = meta["_my_subtitle"] || "";

  return (
    <>
      <PluginSidebarMoreMenuItem target="my-editorial-sidebar">
        {__("Editorial Sidebar", "my-text-domain")}
      </PluginSidebarMoreMenuItem>

      <PluginSidebar
        name="my-editorial-sidebar"
        title={__("Editorial Sidebar", "my-text-domain")}
      >
        <PanelBody
          title={__("Article Settings", "my-text-domain")}
          initialOpen={true}
        >
          <TextControl
            label={__("Subtitle", "my-text-domain")}
            value={value}
            onChange={(next) =>
              editPost({ meta: { ...meta, _my_subtitle: next } })
            }
          />
          <ToggleControl
            label={__("Mark as Featured", "my-text-domain")}
            checked={!!meta["_my_is_featured"]}
            onChange={(checked) =>
              editPost({ meta: { ...meta, _my_is_featured: checked } })
            }
          />
        </PanelBody>
      </PluginSidebar>
    </>
  );
}

registerPlugin("my-editorial-sidebar", {
  render: EditorialSidebar,
  icon: "admin-settings",
});
```

> **Note:** To store data in post meta, register it in PHP via `register_post_meta( $post_type, 'key', [ 'show_in_rest' => true, 'single' => true, 'type' => 'string|boolean|…' ] )` so it’s editable from the editor. ([WordPress Developer Resources][10])

---

### 3) A **PluginDocumentSettingPanel** (panel inside the native Post sidebar)

Use this when the settings **belong to the document itself** (SEO, editorial metadata, etc.), not a free-standing custom sidebar.

```js
const { registerPlugin } = wp.plugins;
const { PluginDocumentSettingPanel } = wp.editor; // WP 6.6+: unified
const { TextControl } = wp.components;
const { useSelect, useDispatch } = wp.data;
const { __ } = wp.i18n;

function SeoDocumentPanel() {
  const meta = useSelect(
    (select) => select("core/editor").getEditedPostAttribute("meta") || {},
    []
  );
  const { editPost } = useDispatch("core/editor");

  return (
    <PluginDocumentSettingPanel
      name="my-seo-panel"
      title={__("SEO (Custom)", "my-text-domain")}
      className="my-seo-panel"
    >
      <TextControl
        label={__("Meta Title", "my-text-domain")}
        value={meta["_seo_title"] || ""}
        onChange={(v) => editPost({ meta: { ...meta, _seo_title: v } })}
      />
      <TextControl
        label={__("Meta Description", "my-text-domain")}
        value={meta["_seo_desc"] || ""}
        onChange={(v) => editPost({ meta: { ...meta, _seo_desc: v } })}
      />
    </PluginDocumentSettingPanel>
  );
}

registerPlugin("my-seo-doc-panel", { render: SeoDocumentPanel });
```

- Panels are **namespaced** by your plugin name (from `registerPlugin`). When toggling them programmatically you must prefix the namespace. 
- When to choose which:

  - **PluginSidebar** → bigger custom area & your own icon in toolbar/⋯ menu.
  - **PluginDocumentSettingPanel** → lives among the built-in “Post/Page” settings. 

---

## Toolbar & menu injections

- `<PluginSidebar/>` automatically adds a toolbar icon **and** a `<PluginSidebarMoreMenuItem/>` in the ⋯ menu to open it. You can also add your own customized More-menu label with `<PluginSidebarMoreMenuItem/>`. 
- You can inject into other editor surfaces via unified SlotFills (6.6+): `PluginBlockSettingsMenuItem`, `PluginPrePublishPanel`, `PluginPostPublishPanel`, etc. Replace any `wp.editPost.*` usage with `wp.editor.*`. ([Make WordPress][3], [GitHub][4])

---

## Real-world use cases that impress interviewers

1. **SEO & Readability**: live character counters, Open Graph preview, “noindex” toggle in a Document panel. (Great `PluginDocumentSettingPanel` fit.) 
2. **Editorial workflow**: a custom sidebar with checklists (“requirements met”), smart defaults (auto‐fill excerpt from first paragraph), and a “Send to Legal” toggle—stored in post meta. 
3. **Pre-publish guardrails**: `PluginPrePublishPanel` that blocks publishing if alt text is missing for images or word count < target. (Use selectors from `core/editor` and show actionable warnings.)
4. **Block-level actions**: `PluginBlockSettingsMenuItem` to run transformations or open custom modals for the selected block. (Unified under `wp.editor` in 6.6.)
5. **Site editor parity**: One code path that appears in both post and site editors thanks to the unified API in 6.6. Less duplication, fewer conditionals.

---

## Quick cheat-sheet

- **Pattern**: Slot (in core) ⇢ Fill (in your plugin).
- **Register**: `wp.plugins.registerPlugin( 'my-namespace', { render, icon } )`.
- **Imports / handles (WP 6.6+)**: Use `wp.editor.*` components (script handle `wp-editor`) for SlotFills; avoid deprecated `wp.editPost.*`. ([Make WordPress][3], [GitHub][4])
- **Popular SlotFills**:

  - `PluginSidebar`, `PluginSidebarMoreMenuItem` (toolbar & ⋯ menu) 
  - `PluginDocumentSettingPanel` (panel inside native doc sidebar) 
  - `PluginPrePublishPanel`, `PluginPostPublishPanel`, `PluginPostStatusInfo`, `PluginBlockSettingsMenuItem` (all unified under `wp.editor`).

- **Data**:

  - Read: `select('core/editor').getEditedPostAttribute('meta')`
  - Write: `dispatch('core/editor').editPost({ meta: { … } })`
  - Ensure `register_post_meta(..., ['show_in_rest' => true])`. ([WordPress Developer Resources][10])

- **WP 6.6 deprecations**: migrate `wp.editPost.*` → `wp.editor.*`. Watch your console for deprecation warnings. ([GitHub][4])
- **Script deps**: include `wp-plugins`, `wp-editor`, `wp-components`, `wp-element`, `wp-data`, `wp-core-data`. ([WordPress Developer Resources][6], [Stack Overflow][9])

---

## Common pitfalls & gotchas (with fixes)

- **Using old namespaces** (`wp.editPost.PluginSidebar`) on WP ≥ 6.6 ⇒ **Fix**: import/use `wp.editor.PluginSidebar` instead. ([GitHub][4])
- **Panel won’t open/toggle** ⇒ Your **panel name must be unique** and **namespaced** per `registerPlugin` docs; when using programmatic toggles, include the namespace. 
- **Missing script dependencies** ⇒ Add `wp-plugins` (for `registerPlugin`) and `wp-editor` (for unified SlotFills). ([WordPress Developer Resources][6], [Stack Overflow][9])
- **Meta not saving** ⇒ You didn’t `register_post_meta` with `show_in_rest: true`. ([WordPress Developer Resources][10])

---

## How Gutenberg “lets you inject UI into the sidebar/toolbar”

1. **Core exposes Slots** in the sidebar area, publish panels, toolbar menus, etc.
2. Your plugin **renders Fills** using helper components like `<PluginSidebar/>` or `<PluginDocumentSettingPanel/>`.
3. You **register** these Fills with `registerPlugin`, and the editor mounts them in the appropriate Slot. The editor also wires conveniences (e.g., a toolbar button + ⋯ menu item for your sidebar). 

---

## Architecture guidance & best practices

- **When to choose Sidebar vs Document Panel**

  - Use **PluginSidebar** for larger, task-oriented experiences (e.g., editorial checklist, asset library).
  - Use **PluginDocumentSettingPanel** for settings that conceptually belong to the document (SEO, canonical URL, noindex). ([rtCamp][11])

- **Unify for WP 6.6+**
  Always import SlotFills from `@wordpress/editor` / `wp.editor` for forward compatibility; remove old `wp.editPost` imports.

- **State & performance**
  Prefer `useEntityProp( 'postType', postType, 'meta' )` (from `@wordpress/core-data`) or `select/dispatch('core/editor')` patterns; avoid ad-hoc REST calls for routine editor state. ([WordPress Developer Resources][10])

- **Accessibility & i18n**
  Use proper labels, roles, and `__()`/`_x()` for strings. ([WordPress Developer Resources][10])

---

## Currently available SlotFills:

(The places you can inject UI) in the Block Editor UI, using the unified `@wordpress/editor` API:

1. **MainDashboardButton** — adds a button to the editor’s top bar (main toolbar).
2. **PluginSidebar** — registers a **custom sidebar** panel; also gets a toolbar toggle automatically.
3. **PluginSidebarMoreMenuItem** — adds an item in the ⋯ **More menu** to open your custom sidebar.
4. **PluginDocumentSettingPanel** — injects a **panel inside the native Document (post/page) sidebar**.
5. **PluginPostStatusInfo** — adds content to the **Post Summary / Status** area in the sidebar.
6. **PluginPrePublishPanel** — surfaces checks/information in the **Pre-publish** panel (before publishing). ([WordPress Developer Resources][3])
7. **PluginPostPublishPanel** — adds content to the **Post-publish** panel (after publishing).
8. **PluginBlockSettingsMenuItem** — inserts a menu item into the **Block settings (ellipsis) menu** for the selected block.
9. **PluginMoreMenuItem** — adds a custom item into the editor’s top-bar **More (⋯) menu**.

> Tip: You can register them **all** inside one `registerPlugin(...)` or keep them separate. I’ll show each one standalone for clarity.

### Examples:

#### 1) MainDashboardButton (top toolbar)

```js
const { MainDashboardButton } = wp.editor;

function MyMainButton() {
  return (
    <MainDashboardButton
      onClick={() => window.alert("Top bar button clicked!")}
      label={__("My Top Button", "td")}
    />
  );
}

registerPlugin("demo-main-button", { render: MyMainButton });
```

#### 2) PluginSidebar (custom sidebar) + 3) PluginSidebarMoreMenuItem (⋯ menu)

```js
const { PluginSidebar, PluginSidebarMoreMenuItem } = wp.editor;
const { PanelBody } = wp.components;

function MySidebar() {
  return (
    <>
      <PluginSidebarMoreMenuItem target="my-sidebar">
        {__("Open My Sidebar", "td")}
      </PluginSidebarMoreMenuItem>

      <PluginSidebar name="my-sidebar" title={__("My Sidebar", "td")}>
        <PanelBody title={__("Settings", "td")} initialOpen>
          <p>{__("Put controls here.", "td")}</p>
        </PanelBody>
      </PluginSidebar>
    </>
  );
}

registerPlugin("demo-plugin-sidebar", {
  render: MySidebar,
  icon: "admin-generic",
});
```

#### 4) PluginDocumentSettingPanel (panel inside Document sidebar)

```js
const { PluginDocumentSettingPanel } = wp.editor;
const { TextControl } = wp.components;
const { useSelect, useDispatch } = wp.data;

function MyDocPanel() {
  const meta = useSelect(
    (s) => s("core/editor").getEditedPostAttribute("meta") || {},
    []
  );
  const { editPost } = useDispatch("core/editor");

  return (
    <PluginDocumentSettingPanel
      name="my-doc-panel"
      title={__("My Doc Panel", "td")}
    >
      <TextControl
        label={__("Subtitle", "td")}
        value={meta._my_subtitle || ""}
        onChange={(v) => editPost({ meta: { ...meta, _my_subtitle: v } })}
      />
    </PluginDocumentSettingPanel>
  );
}

registerPlugin("demo-doc-panel", { render: MyDocPanel });
```

> Don’t forget to register the meta key in PHP with `show_in_rest => true`.

#### 5) PluginPostStatusInfo (extra info under Post Status summary)

```js
const { PluginPostStatusInfo } = wp.editor;
const { useSelect } = wp.data;

function MyPostStatusInfo() {
  const words = useSelect(
    (s) =>
      (s("core/editor").getEditedPostContent() || "")
        .split(/\s+/)
        .filter(Boolean).length,
    []
  );

  return (
    <PluginPostStatusInfo className="my-post-status-info">
      {__("Word count:", "td")} {words}
    </PluginPostStatusInfo>
  );
}

registerPlugin("demo-post-status-info", { render: MyPostStatusInfo });
```

#### 6) PluginPrePublishPanel (before publishing)

```js
const { PluginPrePublishPanel } = wp.editor;
const { useSelect } = wp.data;

function MyPrePublishPanel() {
  const title = useSelect(
    (s) => s("core/editor").getEditedPostAttribute("title") || "",
    []
  );
  const hasTitle = !!title?.trim();

  return (
    <PluginPrePublishPanel title={__("Pre-publish checks", "td")} initialOpen>
      {!hasTitle ? (
        <p style={{ color: "red" }}>
          {__("Add a title before publishing.", "td")}
        </p>
      ) : (
        <p>{__("Looks good!", "td")}</p>
      )}
    </PluginPrePublishPanel>
  );
}

registerPlugin("demo-pre-publish", { render: MyPrePublishPanel });
```

#### 7) PluginPostPublishPanel (after publishing)

```js
const { PluginPostPublishPanel } = wp.editor;
const { useSelect } = wp.data;

function MyPostPublishPanel() {
  const link = useSelect((s) => s("core/editor").getPermalink() || "", []);

  return (
    <PluginPostPublishPanel title={__("Next steps", "td")} initialOpen>
      <p>{__("Share your post:", "td")}</p>
      <a
        href={`https://twitter.com/intent/tweet?url=${encodeURIComponent(
          link
        )}`}
        target="_blank"
        rel="noreferrer"
      >
        {__("Share on X/Twitter", "td")}
      </a>
    </PluginPostPublishPanel>
  );
}

registerPlugin("demo-post-publish", { render: MyPostPublishPanel });
```

#### 8) PluginBlockSettingsMenuItem (selected block’s ⋯ menu)

```js
const { PluginBlockSettingsMenuItem } = wp.editor;
const { useSelect } = wp.data;

function MyBlockMenuItem() {
  const clientIds = useSelect(
    (s) => s("core/block-editor").getSelectedBlockClientIds(),
    []
  );
  const selectedId = clientIds?.[0];

  if (!selectedId) return null;

  return (
    <PluginBlockSettingsMenuItem
      allowedBlocks={["core/paragraph", "core/heading"]} // optional filter
      icon="yes"
      label={__("Say Hi", "td")}
      onClick={() => window.alert("Hello from block menu!")}
    />
  );
}

registerPlugin("demo-block-menu-item", { render: MyBlockMenuItem });
```

#### 9) PluginMoreMenuItem (top bar ⋯ menu)

```js
const { PluginMoreMenuItem } = wp.editor;

function MyMoreMenuItem() {
  return (
    <PluginMoreMenuItem
      icon="smiley"
      onClick={() => window.alert("More Menu clicked")}
    >
      {__("My More Menu Item", "td")}
    </PluginMoreMenuItem>
  );
}

registerPlugin("demo-more-menu-item", { render: MyMoreMenuItem });
```

---

## Interview-style talking points (with strong model answers)

**Q1. What is the SlotFill API and why did Gutenberg adopt it?**
**A.** SlotFill is a React portal/extension pattern where core defines **Slots** and plugins render **Fills** into those slots. It replaced many legacy PHP-based UI hooks to make editor UI extensions predictable, React-native, and compatible with both post and site editors. It supports multiple fills per slot and isolates plugin UI from core render trees. Since WP 6.6, these extension points are unified under `@wordpress/editor`. ([WordPress Developer Resources][1], [Make WordPress][3])

**Q2. Compare `PluginSidebar` and `PluginDocumentSettingPanel`. When would you use each?**
**A.** Use **`PluginSidebar`** when you need a dedicated, togglable sidebar (own toolbar icon/⋯ entry) for workflows like editorial checks or asset pickers. Use **`PluginDocumentSettingPanel`** to add fields inside the native “Post/Page” settings sidebar—for doc-level metadata like SEO title/description, canonical URL, or legal flags. ([WordPress Developer Resources][7], [rtCamp][11])

**Q3. What changed in WordPress 6.6 for extensibility components?**
**A.** 6.6 unified extensibility across post and site editors. SlotFills that used to live in `wp.editPost.*` are **deprecated** in favor of `wp.editor.*` (from `@wordpress/editor`). Updating imports removes console warnings and future-proofs your plugin. ([Make WordPress][3], [GitHub][4])

**Q4. How do you save data from a SlotFill UI?**
**A.** Register post meta with `register_post_meta( ..., [ 'show_in_rest' => true ] )`, then in your SlotFill use `select('core/editor').getEditedPostAttribute('meta')` and `dispatch('core/editor').editPost({ meta })` to read/write. On save, the editor persists post + meta in one flow. ([WordPress Developer Resources][10])

**Q5. What are common pitfalls?**
**A.** The big ones: (1) using deprecated `wp.editPost.*` APIs on WP ≥ 6.6; (2) forgetting to enqueue `wp-plugins` / `wp-editor` dependencies; (3) unregistered or non-REST meta keys; (4) panel name collisions / wrong namespace when toggling. ([GitHub][4], [Stack Overflow][9], [WordPress Developer Resources][10])

**Q6. Can multiple plugins add to the same Slot? How is ordering handled?**
**A.** Yes—Slots accept multiple Fills. Rendering order generally follows mount order (i.e., plugin load sequence). For key areas, Gutenberg provides reasonable defaults; if deterministic ordering matters, coordinate via plugin load order or compose UI so order is less critical.

**Q7. How would you gate UI by post type or capability?**
**A.** In your component, use `select('core/editor').getCurrentPostType()` (or `useSelect`) and `wp.data.select('core').canUser()` to conditionally render fills. You can also conditionally enqueue the script in PHP for specific post types/capabilities.

**Q8. How do you ensure parity between the Post and Site editors?**
**A.** Use the unified `@wordpress/editor` SlotFills (6.6+) and avoid editor-specific conditionals. That way the same code attaches to both editors’ Slots.

---

## References you can cite in an interview

- Slot/Fill conceptual docs and API: **Block Editor Handbook**.
- SlotFills catalog incl. `PluginSidebar`, `PluginDocumentSettingPanel`, etc.
- `@wordpress/plugins` and `registerPlugin` docs.
- WP 6.6 unified extensibility & deprecations: dev note + examples of warnings. ([Make WordPress][3], [GitHub][4])

---

## Slides:

![alt text](<JS Libraries/SlotFill API/SlotFillAPI-1.png>)
![alt text](<JS Libraries/SlotFill API/SlotFillAPI-2.png>)
![alt text](<JS Libraries/SlotFill API/SlotFillAPI-3.png>)
![alt text](<JS Libraries/SlotFill API/SlotFillAPI-4.png>)
![alt text](<JS Libraries/SlotFill API/SlotFillAPI-5.png>)
![alt text](<JS Libraries/SlotFill API/SlotFillAPI-6.png>)
![alt text](<JS Libraries/SlotFill API/SlotFillAPI-7.png>)

---
