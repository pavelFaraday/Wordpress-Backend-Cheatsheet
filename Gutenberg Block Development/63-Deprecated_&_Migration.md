# Deprecated & Migration in Gutenberg Blocks — Convert/Migrate Deprecated code into current/newest version

## Why this matters

When you update a block (change its markup, attributes, or features), old posts may still contain the “old version” of that block. If WordPress can’t recognize it, the block could break (show as HTML or “invalid”).

To avoid this, you declare the old version of your block as “deprecated.”

Gutenberg’s **Deprecation API** lets you define how **_older_** versions of your block are recognized and migrated to the _current_ version safely.

---

## Core concepts

### What “deprecations” are

In `registerBlockType(...)`, you can add a **`deprecated` array**. Each entry describes a _previous_ block version and may include:

- `attributes` — Data schema, the old block version used before.
- `supports` — features the old version used.
- `save` — the old save function (must reproduce the old HTML markup)
- ❗️ `migrate` — how to convert old attributes/innerBlocks to the new shape
- ❗️ `isEligible` — custom test to decide if this deprecation applies

That array is evaluated to recognize and upgrade legacy content when the editor loads it. Declared in **JS registration** (not in `block.json`)❗️.

> So, Each entry item in that array tells WordPress:
>
> - “Here’s what the old version of my block looked like.”
> - “Here’s how to turn/convert old data into the new version.”

#### 🔖 Simple takeaway:

**“Deprecation”** = telling WordPress how your old block versions worked.
**“Migration”** = giving instructions to convert old block data into your new structure.

**`registerBlockType()` Example:**

```js
import { registerBlockType } from "@wordpress/blocks";

registerBlockType("myplugin/notice", {
  title: "Notice",
  category: "widgets",
  attributes: {
    message: { type: "string", default: "Hello World" },
    color: { type: "string", default: "red" },
  },

  edit({ attributes, setAttributes }) {
    return (
      <p
        style={{ color: attributes.color }}
        contentEditable
        onInput={(e) => setAttributes({ message: e.currentTarget.textContent })}
      >
        {attributes.message}
      </p>
    );
  },

  save({ attributes }) {
    return <p style={{ color: attributes.color }}>{attributes.message}</p>;
  },

  // 👇 Deprecated versions of the block
  deprecated: [
    {
      attributes: {
        message: { type: "string", default: "Hello World" },
        color: { type: "string", default: "red" },
      },
      save({ attributes }) {
        // Old version used a CSS class instead of inline style
        return <p className={attributes.color}>{attributes.message}</p>;
      },
      migrate(attributes) {
        // Convert old data (className) → new inline style
        return {
          ...attributes,
          color: attributes.color, // keep color
        };
      },
      isEligible({ color }) {
        // Apply this deprecation only if color looks like old classes
        return ["red", "blue", "green"].includes(color);
      },
    },
  ],
});
```

### How deprecations are applied (important details)

- 📌 Deprecations are **not a chain** of step-by-step migrations. The editor tries each deprecation (usually **most recent first**) until one validates the old markup. That deprecation’s `save` is used to parse attributes; then its `migrate` output (if provided) is fed into your **current** `save`.
- Keep deprecations **stable**: avoid importing helpers that could change over time—snapshot any logic they need.
- Prefer ordering your deprecations **reverse-chronologically** (newest first) to minimize work.

### ⚠️ When _not_ to deprecate

If the new thing is semantically different & completely different idea (or migration is too risky), **publish a new block name** and leave the old one alone. (Users can transform manually if needed.)

##### 🔖 Real case example of Risky Migrations:

- The old “notice” block was just text inside a `<p>`.
- The new “alert” block is semantically different (uses `<div>`, extra attribute type, different styling logic).
- Migration would be too risky (might break user’s expectations).

So, instead of forcing migration, we leave the old block alone (`myplugin/notice`) and ship a new block (`myplugin/alert`).

### Deprecations vs Transforms

- **Deprecations**: keep the _same_ **block name**, upgrade old instances of _this_ block.
  - You keep the same block name.
  - Used when you change how your block works (attributes, markup, etc.), but it’s still “the same block.”
  - You declare the old versions inside `deprecated: []`, so WordPress can recognize them and auto-upgrade them when an editor opens the post.

> 👉 Example:
>
> - Old version saved `<p class="red">Hello</p>`.
> - New version saves `<p style="color:red;">Hello</p>`.
>   → Same block (`myplugin/notice`), just different way of saving > → use deprecations.

- **Transforms**: convert content **between** blocks (e.g., paragraph → heading), or from external forms like shortcodes. Different API (`transforms: { from, to }`).
  - Used when you want to convert between blocks (different names).
  - They are not automatic — the user chooses to transform (via the block toolbar or slash command).
  - Defined with the `transforms` property in your block registration.

> 👉 Example:
>
> - Transform a Paragraph into a Heading.
> - Transform a shortcode into a custom block.
> - Transform your old myplugin/notice block into your new myplugin/alert block.

---

## Migration playbook (practical steps)

1. **Lock the old version**

   - Copy the **exact old `save`** and the **old `attributes`** into a deprecation entry. This must reproduce the legacy markup byte-for-byte (class name differences, wrappers, etc., matter).

2. **Describe the new version normally**

   - Update your block’s main `attributes`, `save`, and (if relevant) `edit`.

3. **Write a `migrate` (only if needed)**

   - If an attribute was renamed, types changed, or inner blocks moved around, return the new shape from `migrate(oldAttributes, oldInnerBlocks)`. (Return `[ newAttributes, newInnerBlocks ]` when inner blocks change.)

4. **Use `isEligible` for ambiguous cases**

   - If the old markup can still validate against multiple shapes, add a precise check so the right deprecation runs.

5. **Order your deprecations**

   - Put the most recent deprecation first; the editor tries them in array order.

6. **Test with real content**

   - Paste HTML from old posts into the editor (or open legacy posts) and confirm:
     a) no “This block contains unexpected or invalid content” banner;
     b) switching to Code Editor shows upgraded markup after resave.

---

## Common migration scenarios & how to approach them

- **Scenario A - Rename an attribute**: In `migrate`, copy the old value to the new key; remove the old key. Keep types consistent.
- **Scenario B - Change attribute type** (string → array, etc.): Convert in `migrate`; ensure the new `save` renders correctly.
- **Scenario C - Markup change** (e.g., `<p>` → `<div>`): Old `save` lives in the deprecation entry so validation passes. No `migrate` needed if attributes didn’t change.
- **Scenario D - Restructure inner blocks**: Use `migrate` to construct a new `innerBlocks` tree and return `[ attributes, innerBlocks ]`.
- **Scenario E - Tighten/relax `supports`**: If old UIs exposed features you’re removing, capture the prior `supports` in the deprecation so parsing remains accurate.

---

## Gotchas (seen in the wild)

- **Markup mismatch** breaks validation:
  Tiny differences (auto-added wrapper classes, tag changes) can cause the deprecation not to match. Ensure the old `save` output is **identical** to what legacy content has, or validation will fail. (There are known issues when wrapper classes differ unexpectedly.)

- **Mutable imports in deprecations:**
  If a deprecation imports helpers that later change, the deprecation silently “drifts.” Prefer **inlining/snapshotting** its logic.

- **Putting `deprecated` in `block.json`** 🚫🙅🏻
  ❗️Deprecations (like `transforms`) belong in the **JS registration**, not in `block.json`❗️

---

## 📌 Quick cheat-sheet

- **Where**: `registerBlockType(..., { deprecated: [ /* entries */ ] })` — **not** in `block.json`.
- **Entry shape**: `{ attributes, supports, save, migrate?, isEligible? }`.
- **Order**: newest first; editor tries each until one validates. Not a chained migration.
- **When to use `migrate`**: attribute rename/type change, innerBlocks restructure. Return `[attrs, innerBlocks]` if you modify children.
- **Alternative**: if the change is radical, publish a **new block name** and skip deprecations.
- **Transforms != deprecations**: transforms convert **between** block types; deprecations upgrade **within** one block.

---

## Use cases in practice

- **Marketing Card v1 → v2**: rename `ctaUrl` → `url` and wrap text in a new container. Keep v1 `save` in the first deprecation; `migrate` copies `ctaUrl` to `url`.
- **List Block layout change**: move list item markup into child blocks. `migrate` builds `innerBlocks` from old text.
- **Design system refresh**: update classes and presets; deprecation saves old HTML for validation while the current `save` emits new class names.

---

## Interview-style talking points (with strong model answers)

**📌 Q1. How does Gutenberg’s deprecation mechanism work?**
**A.** I add a `deprecated` array to `registerBlockType`. Each entry defines a _previous_ version’s `attributes`/`save` and optionally `migrate`/`isEligible`. When loading legacy content, the editor tries each deprecation (newest first) until one validates the saved HTML, then runs its `migrate` to produce attributes/innerBlocks for the **current** `save`. Deprecations aren’t chained; the first match wins.

**📌 Q2. When do you write a `migrate` function vs just supplying old `save`?**
**A.** If only the markup changed, old `save` is enough. If the **schema** changed—renamed/typed attributes or restructured inner blocks—`migrate` maps old data to the new shape (optionally returning `[ attrs, innerBlocks ]`).

**📌 Q3. What’s a reliable workflow to avoid breaking content?**
**A.** Freeze the old version: copy exact old `save` and `attributes` into a deprecation entry; write `migrate` if schema changed; place the deprecation first; test with real legacy posts. Keep deprecation logic self-contained to avoid drift.

**📌 Q4. Deprecations vs transforms—when to use each?**
**A.** Use **deprecations** to upgrade instances of the _same block_. Use **transforms** to convert content across block types or external formats (shortcodes, raw HTML).

**📌 Q5. When would you **not** use deprecations?**
**A.** If the new block is conceptually different or migration is risky, ship/create a **new block name** and optionally provide transforms; that preserves old content without fragile upgrade logic.

**📌 Q6. Name one subtle pitfall you plan for.**
**A.** Sometimes, even a **small difference** in the saved HTML can break a block.
For example:

- If the old version used `<p class="red">Hello</p>`
- And the new version expects `<p style="color:red;">Hello</p>`

👉 Gutenberg might not recognize it as the same block, and it will show an “invalid block” warning.

That’s why, in a **deprecation**, I always make sure the old `save()` function outputs the **exact same HTML** the old block did.
If there are multiple old patterns, I use `isEligible` to check which one applies.

✅ **Simple takeaway:**
Tiny changes in markup can confuse Gutenberg.
To avoid problems, your deprecated `save()` must **match the old HTML exactly**, like a “snapshot” of history.
