# What “filters” are (in Gutenberg JS)

- A **filter** lets you **intercept and change data** before the block editor uses it—very similar to PHP filters.
- Under the hood it’s the `@wordpress/hooks` package (also exposed globally as `wp.hooks`).
- You register callbacks with `addFilter( hookName, namespace, callback, priority? )`.
  Gutenberg triggers those with `applyFilters( hookName, value, ...args )`.

### Filters vs Actions (JS)

- **Filters** transform a value and **must return** it.
- **Actions** just run side effects (no returned value).

---

# Core concepts you’ll be asked about

1. **Hook names are contracts.** You can only hook into names Gutenberg (or plugins) actually `applyFilters`. (e.g., `blocks.registerBlockType`, `editor.BlockEdit`, `blocks.getSaveContent.extraProps`.)
2. **Namespace matters.** Use a unique slug like `my-plugin/feature` so your filter can be removed/overridden predictably.
3. **Priority ordering.** Lower number = earlier. Default is `10`.
4. **Editor lifecycle.** Some filters run at **registration time** (e.g., `blocks.registerBlockType`), others around **render/edit/save** (e.g., `editor.BlockEdit`, `blocks.getSaveContent.extraProps`).
5. **Static vs dynamic blocks.** Filters that change saved markup (e.g., extra props) don’t make sense for blocks that render entirely server-side; prefer editor-side filters for those.
6. **Validation & back-compat.** If you change a block’s saved output for existing posts, you can trigger “This block contains unexpected or invalid content.” Plan migrations or guard conditions.

---

# Most useful JavaScript filters (with practical snippets)

## 1) Modify block settings at registration

**Hook:** `blocks.registerBlockType`
**When:** Right after a block is registered, before it’s used.
**Use cases:** Add attributes, tweak supports, set default variations, enforce constraints.

```js
import { addFilter } from "@wordpress/hooks";

// Add a boolean attribute to several core blocks
addFilter(
  "blocks.registerBlockType",
  "my-plugin/add-flag-attr",
  (settings, name) => {
    const targets = ["core/paragraph", "core/image"];
    if (!targets.includes(name)) return settings;

    return {
      ...settings,
      attributes: {
        ...settings.attributes,
        isFlagged: { type: "boolean", default: false },
      },
      supports: {
        ...settings.supports,
        align: ["wide", "full"], // tighten/expand allowed aligns
      },
    };
  }
);
```

---

## 2) Wrap/extend the **editor UI** for blocks (HOC)

**Hook:** `editor.BlockEdit`
**When:** While rendering the block in the editor.
**Use cases:** Add controls to Inspector, conditionally wrap the edit component, inject context.

```js
import { addFilter } from "@wordpress/hooks";
import { createHigherOrderComponent } from "@wordpress/compose";
import { InspectorControls } from "@wordpress/block-editor";
import { PanelBody, ToggleControl } from "@wordpress/components";

const withFlagControl = createHigherOrderComponent((BlockEdit) => {
  return (props) => {
    if (props.name !== "core/paragraph") return <BlockEdit {...props} />;

    const {
      attributes: { isFlagged },
      setAttributes,
    } = props;

    return (
      <>
        <InspectorControls>
          <PanelBody title="Extra Options">
            <ToggleControl
              label="Flag this paragraph"
              checked={!!isFlagged}
              onChange={(v) => setAttributes({ isFlagged: !!v })}
            />
          </PanelBody>
        </InspectorControls>

        <BlockEdit {...props} />
      </>
    );
  };
}, "withFlagControl");

addFilter("editor.BlockEdit", "my-plugin/with-flag-control", withFlagControl);
```

---

## 3) Add props to the **saved** markup

**Hook:** `blocks.getSaveContent.extraProps`
**When:** Right before Gutenberg serializes a block’s static save output.
**Use cases:** Add classes, `data-*` attributes, ARIA attributes, inline styles.

```js
import { addFilter } from "@wordpress/hooks";

addFilter(
  "blocks.getSaveContent.extraProps",
  "my-plugin/add-save-props",
  (extraProps, blockType, attributes) => {
    if (blockType.name === "core/paragraph" && attributes.isFlagged) {
      extraProps.className = [extraProps.className, "is-flagged"]
        .filter(Boolean)
        .join(" ");
      extraProps["data-flagged"] = "true";
    }
    return extraProps;
  }
);
```

> ⚠️ Changing saved markup on existing content can invalidate blocks; guard with conditions and consider a **Deprecated** version or a migration.

---

## 4) Change the default block CSS class

**Hook:** `blocks.getBlockDefaultClassName`
**When:** When WP generates the default wrapper class (e.g., `wp-block-quote`).
**Use cases:** Standardize classes across themes; integrate with a design system.

```js
import { addFilter } from "@wordpress/hooks";

addFilter(
  "blocks.getBlockDefaultClassName",
  "my-plugin/rename-default-class",
  (className, blockName) => {
    if (blockName === "core/quote") return "c-quote";
    return className;
  }
);
```

---

## 5) Add props to the **editor wrapper** (not saved)

**Hook:** `editor.BlockListBlock`
**When:** While rendering the block container in the editor canvas.
**Use cases:** Add outlines, debug labels, editor-only hints.

```js
import { addFilter } from "@wordpress/hooks";
import { createHigherOrderComponent } from "@wordpress/compose";

const withEditorBadge = createHigherOrderComponent((BlockListBlock) => {
  return (props) => {
    if (props.name !== "core/image") return <BlockListBlock {...props} />;

    const extraProps = {
      ...props,
      className: [props.className, "has-editor-badge"]
        .filter(Boolean)
        .join(" "),
    };

    return <BlockListBlock {...extraProps} />;
  };
}, "withEditorBadge");

addFilter(
  "editor.BlockListBlock",
  "my-plugin/with-editor-badge",
  withEditorBadge
);
```

---

## 6) (Bonus) Replace/augment save element (advanced)

**Hook:** `blocks.getSaveElement` _(available in modern Gutenberg)_
**When:** After a block’s `save()` returns an element, before serialization.
**Use cases:** Wrap saved element, inject children, alter structure (use sparingly).

```js
import { addFilter } from "@wordpress/hooks";
import { createElement } from "@wordpress/element";

addFilter(
  "blocks.getSaveElement",
  "my-plugin/wrap-save-element",
  (element, blockType, attributes) => {
    if (blockType.name !== "core/image") return element;
    return createElement("figure", { className: "img-figure" }, element);
  }
);
```

---

# Real-world mini use cases

1. **Auto-tag external links in Paragraphs**

- Add a boolean attr via `blocks.registerBlockType`.
- UI toggle in `editor.BlockEdit`.
- If on, add `rel="noopener"` and `target="_blank"` on save via `blocks.getSaveContent.extraProps`.

2. **Design system integration**

- Force specific classes via `blocks.getBlockDefaultClassName` and/or `blocks.getSaveContent.extraProps`.
- Disable unsupported aligns by adjusting `supports` in `blocks.registerBlockType`.

3. **Editor-only QA overlays**

- Add grid/outline and block name badge in editor with `editor.BlockListBlock` (no effect on front-end).

---

# Removing / ordering filters

```js
import { addFilter, removeFilter } from "@wordpress/hooks";

// Add with priority 5 (runs earlier than default 10)
addFilter("editor.BlockEdit", "my-plugin/x", cb, 5);

// Later, remove it (e.g., in hot-reload cleanup)
removeFilter("editor.BlockEdit", "my-plugin/x");
```

---

# Testing & safety tips

- **Guard by block name.** Never blanket-modify everything unless intentional.
- **Avoid save-markup churn.** If content already exists in the wild, prefer editor-side changes.
- **Name things uniquely.** `namespace/hook-purpose` avoids collisions.
- **Consider deprecations.** If you must change saved output, add a `deprecated` entry in your block so old markup still parses.

---

# Quick Cheat-Sheet (copy/paste)

```js
// Imports
import { addFilter, removeFilter } from "@wordpress/hooks";
import { createHigherOrderComponent } from "@wordpress/compose";
import { InspectorControls } from "@wordpress/block-editor";
import { PanelBody, ToggleControl } from "@wordpress/components";
import { createElement } from "@wordpress/element";

// 1) Change settings/attributes at registration
addFilter("blocks.registerBlockType", "ns/add-attrs", (settings, name) => {
  if (name !== "core/paragraph") return settings;
  return {
    ...settings,
    attributes: {
      ...settings.attributes,
      isFlagged: { type: "boolean", default: false },
    },
    supports: {
      ...settings.supports,
      align: ["wide", "full"],
    },
  };
});

// 2) Add Inspector control (editor UI)
const withFlagControl = createHigherOrderComponent(
  (BlockEdit) => (props) => {
    if (props.name !== "core/paragraph") return <BlockEdit {...props} />;
    const {
      attributes: { isFlagged },
      setAttributes,
    } = props;
    return (
      <>
        <InspectorControls>
          <PanelBody title="Extra Options">
            <ToggleControl
              label="Flag this paragraph"
              checked={!!isFlagged}
              onChange={(v) => setAttributes({ isFlagged: !!v })}
            />
          </PanelBody>
        </InspectorControls>
        <BlockEdit {...props} />
      </>
    );
  },
  "withFlagControl"
);
addFilter("editor.BlockEdit", "ns/with-flag", withFlagControl);

// 3) Add props to saved markup
addFilter(
  "blocks.getSaveContent.extraProps",
  "ns/save-props",
  (extraProps, blockType, attrs) => {
    if (blockType.name === "core/paragraph" && attrs.isFlagged) {
      extraProps.className = [extraProps.className, "is-flagged"]
        .filter(Boolean)
        .join(" ");
      extraProps["data-flagged"] = "true";
    }
    return extraProps;
  }
);

// 4) Change default class name
addFilter(
  "blocks.getBlockDefaultClassName",
  "ns/default-class",
  (className, blockName) => (blockName === "core/quote" ? "c-quote" : className)
);

// 5) Editor-only wrapper props
const withEditorBadge = createHigherOrderComponent(
  (BlockListBlock) => (props) => {
    if (props.name !== "core/image") return <BlockListBlock {...props} />;
    return (
      <BlockListBlock
        {...props}
        className={[props.className, "has-editor-badge"].join(" ")}
      />
    );
  },
  "withEditorBadge"
);
addFilter("editor.BlockListBlock", "ns/editor-badge", withEditorBadge);

// 6) (Advanced) Wrap the element returned by save()
addFilter("blocks.getSaveElement", "ns/wrap-save", (el, type, attrs) => {
  if (type.name !== "core/image") return el;
  return createElement("figure", { className: "img-figure" }, el);
});
```

---

# Interview-style talking points (quick hits)

- **Why JS filters?** Extend/standardize blocks without forking; compose behavior across many blocks.
- **Top filters you’ve used:** `blocks.registerBlockType`, `editor.BlockEdit`, `editor.BlockListBlock`, `blocks.getSaveContent.extraProps`, `blocks.getBlockDefaultClassName`.
- **Pitfalls:** Invalidation from changed save output; global side-effects; forgetting namespacing; overusing editor filters (perf/UX).
- **Best practice:** Small, focused HOCs; guard by `props.name`; feature-flag via attributes; keep saved changes backward-compatible.

Perfect — let’s expand those **interview-style talking points** into something you could use to sound sharp and confident when asked about Gutenberg JS filters.

## 🎯 Interview-Style Talking Points on Gutenberg JavaScript Filters

### 1. **Purpose & Philosophy**

- _“Filters in Gutenberg are the JavaScript equivalent of PHP filters in WordPress. They let you intercept data at specific hook points and modify it before it’s used by the editor. That means you can change how blocks are registered, rendered, saved, or even styled—without rewriting the block itself.”_
- Highlight: Extensibility → No need to fork core blocks; you can layer custom behavior.

---

### 2. **Core Concepts to Emphasize**

- **Hook system**: Built on `@wordpress/hooks`, same pattern as PHP’s `apply_filters`.
- **Filter vs Action**: Filters must return a value (transformations). Actions don’t.
- **Namespacing**: Always use unique namespace (`my-plugin/feature`) so filters can be removed cleanly.
- **Priority**: Filters execute in order; default = `10`. Useful for chaining logic.
- **Context**: Some filters run at **registration time** (`blocks.registerBlockType`), others during **rendering** (`editor.BlockEdit`) or **saving** (`blocks.getSaveContent.extraProps`).

---

### 3. **Practical Use Cases You Can Drop in**

- **Add new attributes to core blocks** → e.g., add `isFlagged` boolean to Paragraph.
- **Inject custom controls into Inspector** → e.g., a toggle, color picker, or panel.
- **Modify saved markup** → add `data-*` attributes, extra classes, accessibility tags.
- **Standardize design system** → override default block class names.
- **Editor-only enhancements** → outlines, debugging badges, block labels visible only in admin.
- **Marketing/SEO tweaks** → auto-append `rel="nofollow"` to external links or enforce `loading="lazy"` on images.

---

### 4. **Best Practices**

- **Guard filters by block name** — never blindly affect every block unless intentional.
- **Backward compatibility** — changing save output risks block invalidation. Use `deprecated` versions if needed.
- **Keep UI enhancements editor-only** — don’t clutter saved markup unnecessarily.
- **Keep HOCs small & composable** — one filter = one clear responsibility.
- **Test with legacy content** — make sure existing posts don’t break.

---

### 5. **Pitfalls & Tricky Questions**

- **Q:** _“What happens if you change block save markup via a filter?”_
  **A:** It may invalidate blocks in existing posts. To prevent that, you either guard the logic or add deprecated block versions for migration.
- **Q:** _“Can filters slow down the editor?”_
  **A:** Yes, if you wrap too many blocks with heavy HOCs. Always conditionally apply to specific blocks.
- **Q:** _“What’s the difference between `editor.BlockEdit` and `editor.BlockListBlock`?”_
  **A:** `BlockEdit` modifies the block’s actual edit UI; `BlockListBlock` modifies the wrapper container in the editor (useful for outlines or editor-only hints).

---

### 6. **Memorable One-Liners for Interview**

- “Filters are the glue layer that lets plugins extend Gutenberg without forking blocks.”
- “I use `blocks.registerBlockType` for attributes, `editor.BlockEdit` for editor UI, and `blocks.getSaveContent.extraProps` for front-end markup tweaks.”
- “The biggest trap with filters is breaking block validation. Always think about migration paths.”
- “Namespacing and guarding conditions are my golden rules—so my filters don’t step on anyone else’s.”
