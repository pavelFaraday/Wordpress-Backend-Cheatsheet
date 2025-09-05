# Deprecated & Migration in Gutenberg Blocks — Deep Dive + Interview Prep

## Why this matters

Blocks evolve: you’ll rename attributes, adjust markup, or restructure inner blocks. If you ship those changes without a plan, **existing content breaks**. Gutenberg’s **Deprecation API** lets you define how _older_ versions of your block are recognized and migrated to the _current_ version safely. ([WordPress Developer Resources][1])

---

## Core concepts

### What “deprecations” are

In `registerBlockType(...)`, you can provide a **`deprecated` array**. Each entry describes a _previous_ block version and may include:

- `attributes` — schema the old version used
- `supports` — features the old version exposed
- `save` — the old save function (must reproduce the old markup)
- `migrate` — how to convert old attributes/innerBlocks to the new shape
- `isEligible` — custom test to decide if this deprecation applies
  That array is evaluated to recognize and upgrade legacy content when the editor loads it. Declared in **JS registration** (not in `block.json`). ([WordPress Developer Resources][2])

### How deprecations are applied (important details)

- Deprecations are **not a chain** of step-by-step migrations. The editor tries each deprecation (usually **most recent first**) until one validates the old markup. That deprecation’s `save` is used to parse attributes; then its `migrate` output (if provided) is fed into your **current** `save`.
- Keep deprecations **stable**: avoid importing helpers that could change over time—snapshot any logic they need.
- Prefer ordering your deprecations **reverse-chronologically** (newest first) to minimize work. ([WordPress Developer Resources][1])

### When _not_ to deprecate

If the new thing is semantically different (or migration is too risky), **publish a new block name** and leave the old one alone. (Users can transform manually if needed.) ([WordPress Developer Resources][1])

### Deprecations vs Transforms

- **Deprecations**: keep the _same_ block name, upgrade old instances of _this_ block.
- **Transforms**: convert content **between** blocks (e.g., paragraph → heading), or from external forms like shortcodes. Different API (`transforms: { from, to }`). ([WordPress Developer Resources][3])

---

## Migration playbook (practical steps)

1. **Lock the old version**

   - Copy the **exact old `save`** and the **old `attributes`** into a deprecation entry. This must reproduce the legacy markup byte-for-byte (class name differences, wrappers, etc., matter). ([gutenberg.10up.com][4])

2. **Describe the new version normally**

   - Update your block’s main `attributes`, `save`, and (if relevant) `edit`.

3. **Write a `migrate` (only if needed)**

   - If an attribute was renamed, types changed, or inner blocks moved around, return the new shape from `migrate(oldAttributes, oldInnerBlocks)`. (Return `[ newAttributes, newInnerBlocks ]` when inner blocks change.) ([WordPress Developer Resources][1])

4. **Use `isEligible` for ambiguous cases**

   - If the old markup can still validate against multiple shapes, add a precise check so the right deprecation runs. ([WordPress Developer Resources][2])

5. **Order your deprecations**

   - Put the most recent deprecation first; the editor tries them in array order. ([WordPress Developer Resources][1])

6. **Test with real content**

   - Paste HTML from old posts into the editor (or open legacy posts) and confirm:
     a) no “This block contains unexpected or invalid content” banner;
     b) switching to Code Editor shows upgraded markup after resave.

---

## Common migration scenarios & how to approach them

- **Rename an attribute**: In `migrate`, copy the old value to the new key; remove the old key. Keep types consistent. ([gutenberg.10up.com][4])
- **Change attribute type** (string → array, etc.): Convert in `migrate`; ensure the new `save` renders correctly. ([WordPress Developer Resources][1])
- **Markup change** (e.g., `<p>` → `<div>`): Old `save` lives in the deprecation entry so validation passes. No `migrate` needed if attributes didn’t change. ([WordPress Developer Resources][1])
- **Restructure inner blocks**: Use `migrate` to construct a new `innerBlocks` tree and return `[ attributes, innerBlocks ]`. ([WordPress Developer Resources][1])
- **Tighten/relax `supports`**: If old UIs exposed features you’re removing, capture the prior `supports` in the deprecation so parsing remains accurate. ([WordPress Developer Resources][2])

---

## Gotchas (seen in the wild)

- **Markup mismatch** breaks validation
  Tiny differences (auto-added wrapper classes, tag changes) can cause the deprecation not to match. Ensure the old `save` output is **identical** to what legacy content has, or validation will fail. (There are known issues when wrapper classes differ unexpectedly.) ([GitHub][5])

- **Mutable imports in deprecations**
  If a deprecation imports helpers that later change, the deprecation silently “drifts.” Prefer **inlining/snapshotting** its logic. ([WordPress Developer Resources][1])

- **Putting `deprecated` in `block.json`**
  Deprecations (like `transforms`) belong in the **JS registration**, not in `block.json`. ([WordPress Developer Resources][2])

---

## Quick cheat-sheet

- **Where**: `registerBlockType(..., { deprecated: [ /* entries */ ] })` — **not** in `block.json`. ([WordPress Developer Resources][2])
- **Entry shape**: `{ attributes, supports, save, migrate?, isEligible? }`. ([WordPress Developer Resources][2])
- **Order**: newest first; editor tries each until one validates. Not a chained migration. ([WordPress Developer Resources][1])
- **When to use `migrate`**: attribute rename/type change, innerBlocks restructure. Return `[attrs, innerBlocks]` if you modify children. ([WordPress Developer Resources][1])
- **Alternative**: if the change is radical, publish a **new block name** and skip deprecations. ([WordPress Developer Resources][1])
- **Transforms != deprecations**: transforms convert **between** block types; deprecations upgrade **within** one block. ([WordPress Developer Resources][3])

---

## Use cases in practice

- **Marketing Card v1 → v2**: rename `ctaUrl` → `url` and wrap text in a new container. Keep v1 `save` in the first deprecation; `migrate` copies `ctaUrl` to `url`.
- **List Block layout change**: move list item markup into child blocks. `migrate` builds `innerBlocks` from old text.
- **Design system refresh**: update classes and presets; deprecation preserves old HTML for validation while the current `save` emits new class names.

---

## Interview-style talking points (with strong model answers)

**Q1. How does Gutenberg’s deprecation mechanism work?**
**A.** I add a `deprecated` array to `registerBlockType`. Each entry defines a _previous_ version’s `attributes`/`save` and optionally `migrate`/`isEligible`. When loading legacy content, the editor tries each deprecation (newest first) until one validates the saved HTML, then runs its `migrate` to produce attributes/innerBlocks for the **current** `save`. Deprecations aren’t chained; the first match wins. ([WordPress Developer Resources][1])

**Q2. When do you write a `migrate` function vs just supplying old `save`?**
**A.** If only the markup changed, old `save` is enough. If the **schema** changed—renamed/typed attributes or restructured inner blocks—`migrate` maps old data to the new shape (optionally returning `[ attrs, innerBlocks ]`). ([WordPress Developer Resources][1])

**Q3. What’s a reliable workflow to avoid breaking content?**
**A.** Freeze the old version: copy its exact `save` and `attributes` into a deprecation entry; write `migrate` if schema changed; place the deprecation first; test with real legacy posts. Keep deprecation logic self-contained to avoid drift. ([WordPress Developer Resources][1])

**Q4. Deprecations vs transforms—when to use each?**
**A.** Use **deprecations** to upgrade instances of the _same block_. Use **transforms** to convert content across block types or external formats (shortcodes, raw HTML). ([WordPress Developer Resources][3])

**Q5. When would you **not** use deprecations?**
**A.** If the new block is conceptually different or migration is risky, ship a **new block name** and optionally provide transforms; that preserves old content without fragile upgrade logic. ([WordPress Developer Resources][1])

**Q6. Name one subtle pitfall you plan for.**
**A.** Validation can fail due to tiny markup differences (extra wrapper classes, tag changes). I ensure the old `save` mirrors legacy HTML exactly and, if needed, use `isEligible` to disambiguate. ([GitHub][5])
