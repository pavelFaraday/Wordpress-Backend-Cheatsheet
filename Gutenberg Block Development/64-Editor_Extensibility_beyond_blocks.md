# Editor Extensibility _beyond blocks_ (Gutenberg, WP 6+)

Gutenberg gives you multiple levers to change the **editor chrome** itself—not just block UIs. When you work with **Gutenberg (the WordPress block editor)**, you’re not stuck only customizing blocks. You can also customize the _editor interface itself_ — the sidebars, buttons, and options that editors see when creating posts or pages. There are three main ways to do this:

##### 1. **SlotFill (like “plugin slots” in the UI)**

> Drop your own panels/buttons into predefined “slots” (e.g., the document sidebar).

Think of the editor as having empty “slots” in specific places — like the sidebar, toolbar, or footer.
With SlotFill, you can **drop in your own panels or buttons** into these spots.
👉 Example: You add a custom panel in the right sidebar to show “SEO Score” or “Extra Settings.”

##### 2. **Editor Components**

> Ready-made React pieces in `@wordpress/editor` for post-level UI (taxonomies, author, publish, etc.)

WordPress gives you a set of **ready-made React components** (UI building blocks) for common things like categories, tags, author selector, publish date, etc.
👉 Example: Instead of coding your own dropdown to pick a post author, you can just use the built-in `PostAuthor` component from `@wordpress/editor`.

##### 3. **PHP + JS Hooks**

> Filters/actions to tune editor behavior, settings, and which panels appear.

Hooks are like “switches” where you can **change or control how the editor behaves**.

- In PHP: you can hide or show panels.
- In JavaScript: you can filter which options appear.
  👉 Example: Remove the “Discussion” panel if your site doesn’t use comments.

The goal is to extend the editing experience _without_ forking core UI. So, here is the **main idea**: These tools let you **extend or simplify the editor** without rewriting (or “forking”) the WordPress core editor. You just **plug in** your changes.

---

## The two workhorses you’ll use most

#### 1) `PluginDocumentSettingPanel` (SlotFill)

Think of the Document sidebar in Gutenberg (where you normally see things like Categories, Tags, Featured Image). `PluginDocumentSettingPanel` lets you add your own custom panel there. You register it via SlotFill and render any controls inside—great for custom meta or post-level options. So, It looks and feels like a normal WordPress panel (collapsible, draggable, reorderable).
👉 **Example**: You want a “Reading Time” box or “SEO Settings” panel for each post. You use this SlotFill to insert it.

WordPress recently changed the import path `wp.editor.*` (older `wp.editPost.*` still works but is **deprecated** → update your imports).

**Why it’s nice**

- You extend the editor in a way that feels native, instead of hacking together a random UI.
- You can target specific post types or conditions before rendering. For example, choose when it should appear (only for blog posts, or only if the user has a certain role).

#### 2) `PostTaxonomies` (editor component)

Core ready-made component in `@wordpress/editor` that renders the **taxonomy pickers** (Categories/Tags and any CPT taxonomies). If your custom post type has custom taxonomies, it handles them too.
👉 Example: If you add a taxonomy “Genre” for books, PostTaxonomies will automatically show a selector for Genres in the editor.

**It’s smart:** automatically uses dropdowns for flat taxonomies (tags) and checkboxes/tree for hierarchical ones (categories). and exposes subcomponents and a **filter hook** for deep customization.

- You can customize it further with the filter `editor.PostTaxonomyType.`
  The **filter** `editor.PostTaxonomyType` lets you **swap** the UI used for a specific taxonomy (For example, replace checkboxes with radio buttons, or even your own custom selector). This is the supported way to customize taxonomy UI.

**Why it’s nice:** Instead of building taxonomy pickers from scratch, you just reuse WordPress’s own component and optionally tweak it.

✅ Simple takeaway:

- Use `PluginDocumentSettingPanel` when you want to add new panels in the sidebar.

- Use `PostTaxonomies` when you want to show or customize taxonomy selectors.

---

## Modifying the editor interface via filters/actions

### PHP-side editor filters (before the editor loads)

PHP This is where you control what options the editor should have.

- **`block_editor_settings_all`**: 
This filter is like the final checkpoint before settings are sent to the editor.
You can use it to enable/disable features.
👉 Example: Turn off custom colors or restrict font sizes.

- **Post-type supports** (`remove_post_type_support` / `add_post_type_support`): toggling supports (hide/show)
WP Panels (like Excerpt, Featured Image, Discussion) only show up if the post type supports them.
You can add/remove these supports with `add_post_type_support` or `remove_post_type_support`.
👉 Example: If you remove excerpt support from posts, the Excerpt panel disappears from the editor.

### JS-side hooks (once the editor is running)

This is where you can insert your own UI or customize existing ones.

- **SlotFills** you register (e.g., `PluginDocumentSettingPanel`, `PluginSidebar`, pre-/post-publish panels).
- **`editor.PostTaxonomyType`** filter (JS) to replace the taxonomy selector UI per taxonomy.

---

## Practical examples (what teams actually ship)

- **Custom meta panel** (privacy flags, campaign dates, SEO toggles) → `PluginDocumentSettingPanel` + REST-exposed post meta.
- **Replace Categories UI** for a special taxonomy (e.g., choose exactly **one** section via radios) → filter `editor.PostTaxonomyType` to render a custom component.
- **Trim the sidebar** for focused writing → drop supports for comments/trackbacks/formats, and tweak editor settings with `block_editor_settings_all`. The `@wordpress/editor` package exposes the post panels that will auto-disappear when supports are off.

---

## Quick cheat-sheet

- **Add a document panel** → `wp.editor.PluginDocumentSettingPanel` (use `registerPlugin`). `wp.editPost.*` = deprecated alias in 6.6.
- **Customize taxonomy UI** → `@wordpress/editor`’s **`PostTaxonomies`** exists; override per-taxonomy UI with **`editor.PostTaxonomyType`** (JS filter).
- **Hide panels** → remove post-type **supports**; many editor panels render behind `PostTypeSupportCheck`.
- **Global editor knobs** → PHP filter **`block_editor_settings_all`** to adjust settings before the editor loads.
- **6.6 migration** → import SlotFills from **`wp.editor`** to silence deprecation notices.
- ***

## Deep notes: `editor.PostTaxonomies` vs `Editor Document panels`

- **`PostTaxonomies`** (component) renders whatever taxonomies are registered with `show_in_rest: true`. You can also find related helpers (flat/hierarchical selectors, panel wrapper) in the same package.
- **Document settings panels** (SlotFills) are **your** custom panels, not tied to a specific core entity. Use these to surface post meta or controls that don’t belong in a block.

---

---

---

## Interview-style talking points (with strong answers)

**Q1. What’s the difference between extending with SlotFill and using `@wordpress/editor` components?**
**A.** **SlotFill** (e.g., `PluginDocumentSettingPanel`) lets me inject _new_ UI into predefined slots (document sidebar, more menu, pre-publish). **`@wordpress/editor` components** are prebuilt post-level pieces (e.g., `PostTaxonomies`, `PostAuthor`, `PostSchedule`) that I can render or wrap to match post type capabilities. They complement each other: I’ll add a custom panel via SlotFill and populate it with editor components when possible.

**Q2. How do you customize the **Categories/Tags** UI?**
**A.** Use the **`editor.PostTaxonomyType`** filter to replace the default selector for a targeted taxonomy. For example, I can swap the hierarchical checkbox tree for a radio list to enforce a single selection. This is the documented customization point for taxonomy pickers.

**Q3. What changed in WP 6.6 for editor extensibility?**
**A.** SlotFills were **unified**—import from `wp.editor` instead of `wp.editPost`/`wp.editSite`. Old names still render but throw deprecation warnings, so I dual-import or conditionalize for back-compat.

**Q4. How do you remove default panels (e.g., Excerpt, Discussion)?**
**A.** Remove the corresponding **post-type support** on the PHP side. Many editor panels are gated by `PostTypeSupportCheck`, so toggling supports hides UI without hacks. If I need broader tweaks, I filter **`block_editor_settings_all`** before the editor initializes.

**Q5. When do you create a **PluginDocumentSettingPanel** vs a block?**
**A.** Use a **panel** for **post-level metadata** that applies to the entire document (SEO flags, canonical URL, campaign dates). Use a **block** for content that appears in the post body. Panels live in the Document sidebar and don’t serialize into `post_content`.

**Q6. How do you ensure back-compat across WordPress versions?**
**A.** I: (1) import SlotFills from `wp.editor` but keep a fallback to `wp.editPost` for older sites; (2) avoid private APIs; (3) build progressive enhancement around supports (`show_in_rest`, post-type supports) so panels appear only where valid. The 6.6 dev note outlines the migration and dual-support approach.

**Q7. What’s your strategy for custom taxonomy UX in the editor?**
**A.** Keep REST-enabled taxonomies (`show_in_rest: true`) so `PostTaxonomies` can render them; then refine the picker with `editor.PostTaxonomyType` only when business rules demand. I test with large term sets and ensure keyboard navigation works like core.
