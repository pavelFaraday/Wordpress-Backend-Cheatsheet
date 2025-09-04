# Gutenberg Block Development with React in WordPress

### Topics:

1. [From Zero to Block Hero: Mastering Gutenberg Block Development with React in WordPress](1-Gutenberg-React-Basics.md)
2. [How to Build a Custom Gutenberg Block with React (The Ultimate File-by-File Guide)](2-Gutenberg-React-File-structure.md)
3. [Want to Build Custom Gutenberg Blocks? Here’s Your 2025 Guide Using React + Node.js](3-Gutenberg-React-installation-with-nodeJS.md)
4. [Understanding `register_block_type()`](4-register_block_type.md)
5. [Deep Dive into `useBlockProps`](5-useBlockProps.md)
6. [`useInnerBlocksProps` & `InnerBlocks`](6-useInnerBlocksProps-&-InnerBlocks.md)
7. [`Richtext`](7-Richtext.md)
8. [`InspectorControls`](8-InspectorControls.md)
9. [`PanelBody`](9-PanelBody.md)
10. [`SelectControl`](10-SelectControl.md)
11. [`useSelect`](11-useSelect.md)
12. [`registerBlockType`](12-registerBlockType.md)
13. [`BlockControls`](13-BlockControls.md)
14. [`JustifyContentControl`](14-JustifyContentControl.md)
15. [`ToggleControl`](15-ToggleControl.md)
16. [`MediaUploadCheck`](16-MediaUploadCheck.md)
17. [`MediaUpload`](17-MediaUpload.md)
18. [`MediaPlaceholder`](18-MediaPlaceholder.md)
19. [`useState`](19-useState.md)
20. [`ToolbarGroup`](20-ToolbarGroup.md)
21. [`ToolbarButton`](21-ToolbarButton.md)
22. [`Icon`](22-Icon.md)
23. [`AlignmentToolbar`](23-AlignmentToolbar.md)
24. [`ColorPalette`](24-ColorPalette.md)
25. [`RangeControl`](25-RangeControl.md)
26. [`PanelRow`](26-PanelRow.md)
27. [`TextConrol`](27-TextConrol.md)
28. [`useSetting`](28-useSetting.md)
29. [`transforms`](29-transforms.md)
30. [`memoize`](30-memoize.md)
31. [`useDispatch`](31-useDispatch.md)
32. [Static Block vs Dynamic Block](32-static-VS-dynamic.md)
33. [`render-callback` method](33-render-callback.md)
34. [Difference between `title` & `name` in block.json](34-difference_title_&_name.md)
35. [Block Supports API](35-block_supports_API.md)
36. [Wordpress Data Stores](36-wp_data-stores.md)
37. [How WordPress Parses and Renders Post Content](37-wp_post_rendering_parsers_lifeCycle.md)
38. [Block Variations](38-block_variations.md)
39. [Block Example - Demo preview](39-example_demo.md)
40. [Block Transformation in Gutenberg](40-block_transformation.md)
41. [wp.data.subscribe()](41-wp_data_subscribe.md)
42. [wp.data.select()](41-wp-data-select.md)
43. [wp.data.dispatch()](43-wp_data_dispatch.md)
44. [Templates](44-templates.md)
45. [`isBlobURL` in WordPress Gutenberg](45-isBlobURL.md)
46. [`withNotices` - High order components ](46-withNotices.md)
47. [`MediaReplaceFlow` in WordPress Gutenberg](47-MediaReplaceFlow.md)
48. [`useRef` hook in Gutenberg](48-useRef.md)
49. [`usePrevious` hook in Gutenberg](49-usePrevious.md)
50. [Context API in Gutenberg](50-ContextAPI.md)
51. [Predefined Templates & Template Locking in Gutenberg](51-Predefined_Templates+Template_lock.md)
52. [Block Patterns](52-Block-Patterns.md)
53. [PHP Filters for Gutenberg Blocks](53-PHP_filters.md)
54. [JS Filters for Gutenberg Blocks](54-JS_Filters.md)
55. [Internationalization & Localization of Gutenberg Blocks](55-Localization-Translation.md)
56. [SlotFill API (Plugin UI Extensions) in Gutenberg](56-SlotFill-API.md)

---

## 📌 Additional Topics to Strengthen It

### 2. **theme.json & Global Styles** 🎨

- How block supports interact with theme.json.
- Defining color palettes, spacing, typography globally.
- Important for WordPress 6+ era development.

### 3. **Block.json in Detail** 🗂

- Difference between `title`, `name`, `category`, `supports`, `attributes`, `providesContext`, `usesContext`.
- Why `block.json` is central for modern block registration.

### 4. **Block Styles & Variations** 🎭

- You listed “Block Variations” ✅ but also cover **Block Styles** (`registerBlockStyle`).
- Difference between variations vs styles.

### 5. **Server-Side Rendering API (`@wordpress/server-side-render`)** ⚡

- How to preview dynamic blocks in the editor.
- Common for WooCommerce/product-related blocks.

### 6. **Performance & Best Practices** 🚀

- Avoiding unnecessary re-renders with `useMemo`, `useCallback`.
- Splitting code into smaller components.
- Accessibility (focus handling, ARIA roles).

### 7. **Security & Sanitization** 🔒

- Escaping and sanitizing block attributes in PHP (`esc_html`, `esc_url`).
- Avoiding XSS with `dangerouslySetInnerHTML`.

### 8. **Deprecated & Migration** 🔄

- Handling block deprecations (the `deprecated` property in block registration).
- Migrating attributes when schema changes.

### 9. **Testing Gutenberg Blocks** 🧪

- Unit tests (Jest, React Testing Library).
- End-to-end tests with Playwright/Puppeteer + WP test utils.

### 10. **Editor Extensibility Beyond Blocks** 🖇

- Modifying the editor interface via filters/actions.
- Example: `editor.PostTaxonomies`, `editor.DocumentSettingsPanel`.

---

## ✅ Suggested Additions (Quick Checklist)

- SlotFill API (`PluginSidebar`, `PluginDocumentSettingPanel`)
- theme.json & Global Styles
- Block Styles (`registerBlockStyle`)
- Server-Side Rendering (SSR)
- Block Deprecation & Migration
- Testing Blocks (unit + e2e)
- Security & Sanitization
- Performance optimizations (React hooks, memoization best practices)

---
