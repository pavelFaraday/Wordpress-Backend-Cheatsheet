# What “Context” is (in Gutenberg)

**Block Context** lets a parent (ancestor) block expose values that any descendant block can read—without prop-drilling via attributes or hard-coding. Think React Context, but built into the Block API and available to both the editor (JS) and dynamic rendering (PHP). ([WordPress Developer Resources][1])

Key traits:

- Defined declaratively in your block’s settings/`block.json` using **`providesContext`** (parent) and **`usesContext`** (child). ([WordPress Developer Resources][1])
- Values come from the provider block’s **attributes** (for now)—you map a context key to an attribute. Namespacing the key is recommended (e.g., `"my-plugin/recordId"`). ([WordPress Developer Resources][1])
- Accessible in **`edit({ context })`** (JS) and in **`$block->context`** inside **`render_callback`** (PHP). **Not available in `save()`**. ([WordPress Developer Resources][1])

# When to use it

- **Parent controls child behavior**: e.g., a “Card Grid” block supplies `columns`/`accentColor` to inner “Card” blocks.
- **Query Loop descendants**: child blocks inside **Query Loop** should read **`postId`**/`postType`/`queryId` from context rather than from `core/editor` (which would give the editor’s current post, not the loop item). ([WordPress Development Stack Exchange][2], [Misha Rudrastyh][3])
- **Theme / FSE templates**: parent template blocks provide post-aware context (e.g., showing the excerpt for whatever post the template is iterating). ([WordPress Developer Resources][1])
- **ACF Blocks**: ACF 6+ supports WordPress block context if you’re building blocks with ACF. ([ACF][4])

# How it works: step by step

## 1) Parent provides context (block.json)

```json
{
  "apiVersion": 3,
  "name": "acme/card-grid",
  "attributes": {
    "columns": { "type": "number", "default": 3 },
    "accentColor": { "type": "string", "default": "#0ea5e9" }
  },
  "providesContext": {
    "acme/columns": "columns",
    "acme/accentColor": "accentColor"
  },
  "supports": { "anchor": true },
  "editorScript": "file:./index.js"
}
```

Context keys are arbitrary strings; **namespace them** (e.g., `acme/*`) to avoid collisions. The values must map to attributes. ([WordPress Developer Resources][1])

## 2) Child consumes context (block.json)

```json
{
  "apiVersion": 3,
  "name": "acme/card",
  "usesContext": ["acme/columns", "acme/accentColor"],
  "editorScript": "file:./index.js"
}
```

`usesContext` is just an array of the context keys the block opts into. ([WordPress Developer Resources][1])

## 3) Read it in the editor (JS)

```js
import { registerBlockType } from "@wordpress/blocks";
import { useBlockProps } from "@wordpress/block-editor";

registerBlockType("acme/card", {
  edit({ context }) {
    const cols = context["acme/columns"]; // number
    const color = context["acme/accentColor"]; // string
    return (
      <div {...useBlockProps()} style={{ borderColor: color }}>
        <p>Rendered inside a {cols}-column grid.</p>
      </div>
    );
  },
  save() {
    // Note: Block Context is NOT available here.
    return null;
  },
});
```

Context is passed into `edit({ context })`. **You won’t see it in `save()`**—use dynamic rendering if you need it on the front end. ([WordPress Developer Resources][1])

## 4) Use it in dynamic PHP rendering

```php
register_block_type( 'acme/card', array(
  'render_callback' => function( $attributes, $content, $block ) {
    $cols  = $block->context['acme/columns'] ?? 3;
    $color = $block->context['acme/accentColor'] ?? '#0ea5e9';
    return sprintf(
      '<div class="acme-card cols-%d" style="--accent:%s">%s</div>',
      intval( $cols ),
      esc_attr( $color ),
      $content
    );
  },
) );
```

Context lives on the `$block->context` array in `render_callback`. ([WordPress Developer Resources][1])

# Real-world examples you’ll run into

- **Query Loop → Post blocks**: Post Title / Excerpt / Featured Image inside a Query Loop should read the loop’s **`postId`**/**`postType`** via context, not the editor’s post. This avoids wrong data when previewing nested content in templates/patterns. ([WordPress Development Stack Exchange][2])
- **Navigation**: Core Nav provides context to certain descendants so they know they’re inside a navigation structure (useful for variations/extensions). ([GitHub][5])
- **ACF blocks**: parent ACF block can provide context that inner ACF or native blocks use—handy for shared settings like “theme” or “layout mode.” ([ACF][4])

# Gotchas & best practices

- **Opt-in both sides**: Provider must define `providesContext`, consumer must declare `usesContext`, or you’ll get `undefined`. ([WordPress Developer Resources][1])
- **No `save()` access**: If front-end needs context, make the consumer a **dynamic** block and use PHP. ([WordPress Developer Resources][1])
- **Attributes-only sources**: You can only map from provider **attributes** (today). If you need derived values, compute them into an attribute (or compute in PHP). ([WordPress Developer Resources][1])
- **Scope is hierarchical**: Only descendants in the same block tree get it. A nested provider can **override** an ancestor’s value by providing the same key. ([WordPress Developer Resources][1])
- **Namespacing matters**: Use `my-plugin/*` keys to avoid collisions with core or other plugins. ([WordPress Developer Resources][1])
- **Query Loop specifics**: Don’t use `core/editor`’s current post selectors in Query Loop children—use context (`postId`, `postType`, `queryId`, etc.). ([WordPress Development Stack Exchange][2], [Misha Rudrastyh][3])

# Minimal end-to-end example (Context + InnerBlocks)

**Parent** provides an ID that children show:

```js
// parent: my-plugin/record
registerBlockType("my-plugin/record", {
  attributes: { recordId: { type: "number" } },
  providesContext: { "my-plugin/recordId": "recordId" },
  edit({ attributes, setAttributes }) {
    return (
      <>
        <input
          type="number"
          value={attributes.recordId || ""}
          onChange={(e) => setAttributes({ recordId: Number(e.target.value) })}
        />
        <InnerBlocks allowedBlocks={["my-plugin/record-title"]} />
      </>
    );
  },
  save() {
    return null;
  },
});

// child: my-plugin/record-title
registerBlockType("my-plugin/record-title", {
  usesContext: ["my-plugin/recordId"],
  edit({ context }) {
    return <p>ID in editor: {context["my-plugin/recordId"]}</p>;
  },
  save() {
    return null;
  },
});
```

The same structure appears in the official docs’ example, including the PHP `render_callback` usage. ([WordPress Developer Resources][1])

---

# Quick Cheat-Sheet

**Define (parent)**

- `block.json` →
  `"providesContext": { "acme/flag": "flagAttr" }`
  _(maps a context key to a provider attribute)_ ([WordPress Developer Resources][1])

**Consume (child)**

- `block.json` →
  `"usesContext": ["acme/flag"]` ([WordPress Developer Resources][1])

**Editor (JS)**

- `edit({ context })` → `context['acme/flag']` ([WordPress Developer Resources][1])

**Frontend (PHP, dynamic)**

- `render_callback( $attrs, $content, $block )` → `$block->context['acme/flag'] ?? null` ([WordPress Developer Resources][1])

**Remember**

- Namespacing keys (`acme/*`)
- Not available in `save()`
- Great for Query Loop descendants (`postId`, `postType`, `queryId`) ([WordPress Development Stack Exchange][2], [Misha Rudrastyh][3])
- ACF Blocks support context too. ([ACF][4])

[1]: https://developer.wordpress.org/block-editor/reference-guides/block-api/block-context/ "Context – Block Editor Handbook | Developer.WordPress.org"
[2]: https://wordpress.stackexchange.com/questions/427206/block-that-displays-post-meta-as-nested-block-of-query-loop?utm_source=chatgpt.com "Block that displays post meta as nested block of Query Loop"
[3]: https://rudrastyh.com/wordpress/query-loop-block.html?utm_source=chatgpt.com "WordPress Query Loop Block Tutorial"
[4]: https://www.advancedcustomfields.com/resources/using-context-with-acf-blocks/?utm_source=chatgpt.com "Using Context With ACF Blocks"
[5]: https://github.com/WordPress/gutenberg/issues/58822?utm_source=chatgpt.com "Provide parent block as context to `render_callback` of block"

---

---

---

# 📦 Project Example: `acme-context-demo`

**Goal:** Parent passes `currency`, `billingPeriod`, and `accentColor` to children via **context**.
**Tech:** Dynamic blocks (PHP render), ES5 editor scripts, `block.json` metadata.

```
acme-context-demo/
├─ acme-context-demo.php
└─ blocks/
   ├─ pricing-table/
   │  ├─ block.json
   │  ├─ index.js
   │  └─ style.css
   └─ pricing-item/
      ├─ block.json
      ├─ index.js
      └─ style.css
```

---

## 1) Plugin bootstrap (`acme-context-demo.php`)

```php
<?php
/**
 * Plugin Name: ACME Context Demo
 * Description: Minimal example of Gutenberg Block Context (parent provides, child consumes).
 * Version:     1.0.0
 * Author:      You
 */

if ( ! defined( 'ABSPATH' ) ) exit;

function acme_context_demo_register_blocks() {
	$base = __DIR__ . '/blocks';

	// Parent: pricing-table (provides context)
	register_block_type_from_metadata( $base . '/pricing-table', array(
		'render_callback' => function( $attrs, $content, $block ) {
			$color  = isset( $attrs['accentColor'] ) ? esc_attr( $attrs['accentColor'] ) : '#0ea5e9';
			$period = isset( $attrs['billingPeriod'] ) ? esc_html( $attrs['billingPeriod'] ) : 'month';
			$curr   = isset( $attrs['currency'] ) ? esc_html( $attrs['currency'] ) : '$';

			return sprintf(
				'<div class="acme-pricing-table" style="--acme-accent:%1$s" data-period="%2$s" data-currency="%3$s">%4$s</div>',
				$color,
				$period,
				$curr,
				$content // InnerBlocks markup
			);
		},
	) );

	// Child: pricing-item (uses context)
	register_block_type_from_metadata( $base . '/pricing-item', array(
		'render_callback' => function( $attrs, $content, $block ) {
			$ctx     = isset( $block->context ) ? (array) $block->context : array();
			$curr    = isset( $ctx['acme/currency'] ) ? $ctx['acme/currency'] : '$';
			$period  = isset( $ctx['acme/billingPeriod'] ) ? $ctx['acme/billingPeriod'] : 'month';
			$color   = isset( $ctx['acme/accentColor'] ) ? $ctx['acme/accentColor'] : '#0ea5e9';

			$title   = isset( $attrs['title'] ) ? esc_html( $attrs['title'] ) : 'Plan';
			$price   = isset( $attrs['price'] ) ? esc_html( $attrs['price'] ) : '0';
			$badge   = ! empty( $attrs['featured'] ) ? '<span class="acme-badge">Popular</span>' : '';

			return sprintf(
				'<div class="acme-pricing-item" style="border-color:%1$s">
					%2$s
					<h3>%3$s</h3>
					<div class="acme-price"><span class="acme-curr">%4$s</span>%5$s <small>/ %6$s</small></div>
					<div class="acme-body">%7$s</div>
				</div>',
				esc_attr( $color ),
				$badge,
				$title,
				esc_html( $curr ),
				esc_html( $price ),
				esc_html( $period ),
				$content // InnerBlocks (e.g., list of features)
			);
		},
	) );
}
add_action( 'init', 'acme_context_demo_register_blocks' );
```

---

## 2) Parent block: `blocks/pricing-table/block.json`

```json
{
  "apiVersion": 3,
  "name": "acme/pricing-table",
  "title": "Pricing Table (Parent)",
  "category": "design",
  "icon": "table-col-after",
  "description": "Parent block that provides currency, billing period, and accent color to its children.",
  "supports": { "anchor": true },
  "attributes": {
    "currency": { "type": "string", "default": "$" },
    "billingPeriod": { "type": "string", "default": "month" },
    "accentColor": { "type": "string", "default": "#0ea5e9" }
  },
  "providesContext": {
    "acme/currency": "currency",
    "acme/billingPeriod": "billingPeriod",
    "acme/accentColor": "accentColor"
  },
  "editorScript": "file:index.js",
  "style": "file:style.css"
}
```

### `blocks/pricing-table/index.js` (ES5, no build)

```js
(function (wp) {
  var el = wp.element.createElement;
  var __ = wp.i18n.__;
  var useBlockProps = wp.blockEditor.useBlockProps;
  var InspectorControls = wp.blockEditor.InspectorControls;
  var PanelBody = wp.components.PanelBody;
  var TextControl = wp.components.TextControl;
  var ColorPalette = wp.blockEditor.ColorPalette;
  var InnerBlocks = wp.blockEditor.InnerBlocks;

  wp.blocks.registerBlockType("acme/pricing-table", {
    edit: function (props) {
      var a = props.attributes;
      var set = props.setAttributes;

      return el(
        "div",
        useBlockProps({ className: "acme-pricing-table-editor" }),
        el(
          InspectorControls,
          {},
          el(
            PanelBody,
            { title: __("Table Settings", "acme"), initialOpen: true },
            el(TextControl, {
              label: __("Currency Symbol", "acme"),
              value: a.currency,
              onChange: function (v) {
                set({ currency: v });
              },
            }),
            el(TextControl, {
              label: __("Billing Period (e.g. month, year)", "acme"),
              value: a.billingPeriod,
              onChange: function (v) {
                set({ billingPeriod: v });
              },
            }),
            el(
              "div",
              { style: { marginTop: "8px" } },
              __("Accent Color", "acme")
            ),
            el(ColorPalette, {
              value: a.accentColor,
              onChange: function (v) {
                set({ accentColor: v });
              },
            })
          )
        ),
        el(
          "div",
          { className: "acme-pricing-table-help" },
          __(
            "Add “Pricing Item” blocks inside. They’ll auto-pick currency/period/color from here.",
            "acme"
          )
        ),
        el(InnerBlocks, {
          allowedBlocks: ["acme/pricing-item"],
          orientation: "horizontal",
        })
      );
    },
    save: function () {
      return null;
    }, // dynamic
  });
})(window.wp);
```

---

## 3) Child block: `blocks/pricing-item/block.json`

```json
{
  "apiVersion": 3,
  "name": "acme/pricing-item",
  "title": "Pricing Item (Child)",
  "category": "design",
  "icon": "index-card",
  "description": "Child card that consumes currency, billing period, and accent color from the parent.",
  "supports": { "anchor": true },
  "attributes": {
    "title": { "type": "string", "default": "Starter" },
    "price": { "type": "string", "default": "19" },
    "featured": { "type": "boolean", "default": false }
  },
  "usesContext": ["acme/currency", "acme/billingPeriod", "acme/accentColor"],
  "editorScript": "file:index.js",
  "style": "file:style.css"
}
```

### `blocks/pricing-item/index.js` (ES5, no build)

```js
(function (wp) {
  var el = wp.element.createElement;
  var __ = wp.i18n.__;
  var useBlockProps = wp.blockEditor.useBlockProps;
  var InspectorControls = wp.blockEditor.InspectorControls;
  var PanelBody = wp.components.PanelBody;
  var TextControl = wp.components.TextControl;
  var ToggleControl = wp.components.ToggleControl;

  wp.blocks.registerBlockType("acme/pricing-item", {
    edit: function (props) {
      var a = props.attributes;
      var set = props.setAttributes;
      var ctx = props.context || {};
      var curr = ctx["acme/currency"] || "$";
      var period = ctx["acme/billingPeriod"] || "month";

      return el(
        "div",
        useBlockProps({ className: "acme-pricing-item-editor" }),
        el(
          InspectorControls,
          {},
          el(
            PanelBody,
            { title: __("Item Settings", "acme"), initialOpen: true },
            el(TextControl, {
              label: __("Title", "acme"),
              value: a.title,
              onChange: function (v) {
                set({ title: v });
              },
            }),
            el(TextControl, {
              label: __("Price (number or text)", "acme"),
              value: a.price,
              onChange: function (v) {
                set({ price: v });
              },
            }),
            el(ToggleControl, {
              label: __("Featured (adds badge)", "acme"),
              checked: !!a.featured,
              onChange: function (v) {
                set({ featured: !!v });
              },
            })
          )
        ),
        el(
          "div",
          { className: "acme-pricing-item-preview" },
          el(
            "div",
            { className: "acme-badge" },
            a.featured ? __("Popular", "acme") : ""
          ),
          el("h3", null, a.title),
          el(
            "div",
            { className: "acme-price" },
            el("span", { className: "acme-curr" }, curr),
            a.price,
            el("small", null, " / " + period)
          ),
          el(wp.blockEditor.InnerBlocks, {
            placeholder: __(
              "Add features (list, paragraphs, buttons...)",
              "acme"
            ),
          })
        )
      );
    },
    save: function () {
      return null;
    }, // dynamic
  });
})(window.wp);
```

---

## 4) How to use (quick)

1. Drop the folder into `wp-content/plugins/acme-context-demo` and **Activate** the plugin.
2. In the editor, insert **“Pricing Table (Parent)”**.
3. Set **Currency**, **Billing Period**, and **Accent Color** in the sidebar.
4. Add a few **“Pricing Item (Child)”** blocks inside; they’ll automatically display with the parent’s currency/period/color—**no prop-drilling**.
5. Add lists/buttons inside each item (they render as `InnerBlocks` content).

---

## 5) What this demonstrates (context in practice)

- **Parent → Child data flow** via `providesContext` / `usesContext`.
- **Editor consumption**: `edit( { context } )` (we preview with the right currency/period).
- **Frontend consumption**: `$block->context[...]` in `render_callback` (dynamic, accurate on the site).
- **Namespaced keys** (`acme/*`) to avoid collisions.

---
