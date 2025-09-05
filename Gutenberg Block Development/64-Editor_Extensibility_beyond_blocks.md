# Editor Extensibility *beyond blocks* (Gutenberg, WP 6+)

Gutenberg gives you multiple levers to change the **editor chrome** itself—not just block UIs. Three big buckets:

1. **SlotFill**: drop your own panels/buttons into predefined “slots” (e.g., the document sidebar).
2. **Editor components**: ready-made React pieces in `@wordpress/editor` for post-level UI (taxonomies, author, publish, etc.).
3. **PHP + JS hooks**: filters/actions to tune editor behavior, settings, and which panels appear.

The goal is to extend the editing experience *without* forking core UI. ([WordPress Developer Resources][1])

---

## The two workhorses you’ll use most

### 1) `PluginDocumentSettingPanel` (SlotFill)

Adds a **new panel** in the **Document** sidebar (same column as Categories/Tags, Featured image, etc.). You register it via SlotFill and render any controls inside—great for custom meta or post-level options. Since WP 6.6 the SlotFills were **unified** under `wp.editor.*` (older `wp.editPost.*` still works but is **deprecated** → update your imports). ([WordPress Developer Resources][2], [Make WordPress][3], [GitHub][4])

**Why it’s nice**

* Looks and behaves exactly like native panels (collapsible, reorderable).
* You can target specific post types or conditions before rendering.
  Docs: *PluginDocumentSettingPanel* in the Block Editor Handbook. ([WordPress Developer Resources][2])

---

### 2) `PostTaxonomies` (editor component)

Core component in `@wordpress/editor` that renders the **taxonomy pickers** (Categories/Tags and any CPT taxonomies). It chooses flat vs hierarchical selectors automatically and exposes subcomponents and a **filter hook** for deep customization. ([WordPress Developer Resources][5])

* The **filter** `editor.PostTaxonomyType` lets you **swap** the UI used for a specific taxonomy (e.g., render radios instead of checkboxes, or your own selector). This is the supported way to customize taxonomy UI. ([GitHub][6], [WordPress Development Stack Exchange][7])

---

## Modifying the editor interface via filters/actions

### PHP-side editor filters (before the editor boots)

* **`block_editor_settings_all`**: last-mile place to tweak settings sent to the editor (feature flags, allowed features, etc.). Prior to WP 5.8 this was `block_editor_settings`. ([GitHub][8])
* **Post-type supports** (`remove_post_type_support` / `add_post_type_support`): toggling supports hides/shows panels because many editor components are wrapped in **`PostTypeSupportCheck`**. (Example: removing `excerpt` support drops the Excerpt panel.) ([WordPress Developer Resources][5])

### JS-side hooks

* **SlotFills** you register (e.g., `PluginDocumentSettingPanel`, `PluginSidebar`, pre-/post-publish panels). In WP 6.6+ import from `wp.editor` to avoid deprecation warnings. ([Make WordPress][3], [GitHub][4])
* **`editor.PostTaxonomyType`** filter (JS) to replace the taxonomy selector UI per taxonomy. ([GitHub][6])

> Heads-up on WP 6.6: “Unified Extensibility APIs” moved common SlotFills under `wp.editor`. `wp.editPost.*` still works but logs deprecations—update your imports for a clean console. ([Make WordPress][3], [GenerateSupport][9])

---

## Practical examples (what teams actually ship)

* **Custom meta panel** (privacy flags, campaign dates, SEO toggles) → `PluginDocumentSettingPanel` + REST-exposed post meta. ([WordPress Developer Resources][2])
* **Replace Categories UI** for a special taxonomy (e.g., choose exactly **one** section via radios) → filter `editor.PostTaxonomyType` to render a custom component. ([GitHub][6])
* **Trim the sidebar** for focused writing → drop supports for comments/trackbacks/formats, and tweak editor settings with `block_editor_settings_all`. The `@wordpress/editor` package exposes the post panels that will auto-disappear when supports are off. ([WordPress Developer Resources][5], [GitHub][8])

---

## Quick cheat-sheet

* **Add a document panel** → `wp.editor.PluginDocumentSettingPanel` (use `registerPlugin`). `wp.editPost.*` = deprecated alias in 6.6. ([WordPress Developer Resources][2], [Make WordPress][3])
* **Customize taxonomy UI** → `@wordpress/editor`’s **`PostTaxonomies`** exists; override per-taxonomy UI with **`editor.PostTaxonomyType`** (JS filter). ([WordPress Developer Resources][5], [GitHub][6])
* **Hide panels** → remove post-type **supports**; many editor panels render behind `PostTypeSupportCheck`. ([WordPress Developer Resources][5])
* **Global editor knobs** → PHP filter **`block_editor_settings_all`** to adjust settings before the editor loads. ([GitHub][8])
* **6.6 migration** → import SlotFills from **`wp.editor`** to silence deprecation notices. ([Make WordPress][3], [GitHub][4])

---

## Deep notes: `editor.PostTaxonomies` vs `Editor Document panels`

* **`PostTaxonomies`** (component) renders whatever taxonomies are registered with `show_in_rest: true`. You can also find related helpers (flat/hierarchical selectors, panel wrapper) in the same package. ([WordPress Developer Resources][5])
* **Document settings panels** (SlotFills) are **your** custom panels, not tied to a specific core entity. Use these to surface post meta or controls that don’t belong in a block. ([WordPress Developer Resources][2])

---

## Interview-style talking points (with strong answers)

**Q1. What’s the difference between extending with SlotFill and using `@wordpress/editor` components?**
**A.** **SlotFill** (e.g., `PluginDocumentSettingPanel`) lets me inject *new* UI into predefined slots (document sidebar, more menu, pre-publish). **`@wordpress/editor` components** are prebuilt post-level pieces (e.g., `PostTaxonomies`, `PostAuthor`, `PostSchedule`) that I can render or wrap to match post type capabilities. They complement each other: I’ll add a custom panel via SlotFill and populate it with editor components when possible. ([WordPress Developer Resources][2])

**Q2. How do you customize the **Categories/Tags** UI?**
**A.** Use the **`editor.PostTaxonomyType`** filter to replace the default selector for a targeted taxonomy. For example, I can swap the hierarchical checkbox tree for a radio list to enforce a single selection. This is the documented customization point for taxonomy pickers. ([GitHub][6])

**Q3. What changed in WP 6.6 for editor extensibility?**
**A.** SlotFills were **unified**—import from `wp.editor` instead of `wp.editPost`/`wp.editSite`. Old names still render but throw deprecation warnings, so I dual-import or conditionalize for back-compat. ([Make WordPress][3], [GitHub][4])

**Q4. How do you remove default panels (e.g., Excerpt, Discussion)?**
**A.** Remove the corresponding **post-type support** on the PHP side. Many editor panels are gated by `PostTypeSupportCheck`, so toggling supports hides UI without hacks. If I need broader tweaks, I filter **`block_editor_settings_all`** before the editor initializes. ([WordPress Developer Resources][5], [GitHub][8])

**Q5. When do you create a **PluginDocumentSettingPanel** vs a block?**
**A.** Use a **panel** for **post-level metadata** that applies to the entire document (SEO flags, canonical URL, campaign dates). Use a **block** for content that appears in the post body. Panels live in the Document sidebar and don’t serialize into `post_content`. ([WordPress Developer Resources][2])

**Q6. How do you ensure back-compat across WordPress versions?**
**A.** I: (1) import SlotFills from `wp.editor` but keep a fallback to `wp.editPost` for older sites; (2) avoid private APIs; (3) build progressive enhancement around supports (`show_in_rest`, post-type supports) so panels appear only where valid. The 6.6 dev note outlines the migration and dual-support approach. ([Make WordPress][3])

**Q7. What’s your strategy for custom taxonomy UX in the editor?**
**A.** Keep REST-enabled taxonomies (`show_in_rest: true`) so `PostTaxonomies` can render them; then refine the picker with `editor.PostTaxonomyType` only when business rules demand. I test with large term sets and ensure keyboard navigation works like core. ([WordPress Developer Resources][5], [GitHub][6])

