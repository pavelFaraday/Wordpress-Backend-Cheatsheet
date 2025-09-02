# Predefined Templates & Template Locking in Gutenberg (Deep-dive + mini project + cheat-sheet)

## Why they exist (the purpose)

Predefined **post type templates** and **template locking** give you:

* **Editorial guardrails** – every new post starts with the same structure.
* **Brand consistency** – authors can edit content but not break the layout.
* **Speed** – no need to rebuild the same sections each time.
* **Safer collaboration** – you decide what can be inserted, moved, or removed.

---

## Core concepts (clear mental model)

### 1) Post type editor templates (PHP)

A **template** is an array that describes the **initial block tree** for a post type. It’s applied **only when creating a new post** for that type.

Shape (pseudocode):

```php
$template = [
  [ 'block/name',           [ attributes... ], [ innerBlocks... ] ],
  [ 'core/paragraph',       [ 'content' => 'Hello' ] ],
  [ 'core/columns',         [], [
      [ 'core/column', [], [ [ 'core/paragraph', [] ] ] ],
      [ 'core/column', [], [ [ 'core/image', [] ] ] ],
  ]],
];
```

Where to set:

* In `register_post_type( ..., [ 'template' => $template, 'template_lock' => 'all' ] )`
* Or mutate an existing type (e.g., `post`) via `get_post_type_object( 'post' )` inside `init` and set `$pto->template` / `$pto->template_lock`.

**Important gotchas**

* Requires `show_in_rest => true` for the post type (Gutenberg editor).
* Applies to **new** posts only (doesn’t retrofit existing content).
* If custom blocks are referenced, they **must be registered** before the editor loads.

### 2) Template locking (what is locked?)

Controls **structure** editing, not content editing itself.

At **post type** level (`template_lock` in PHP):

* `false` (default): free editing
* `'insert'`: can move/remove existing blocks, but **cannot insert new** ones
* `'all'`: cannot insert, move, or remove blocks

At **block layout** level (JS, with `<InnerBlocks>`):

* `templateLock="false" | "insert" | "all" | "contentOnly"`

  * `'contentOnly'` (InnerBlocks only): users edit **text/media only**; block UI chrome is hidden.

At **individual block** level (UI/attributes):

* Block attribute `lock` can disallow `move`/`remove`.
* Users with permission can also lock blocks via the editor UI (three-dot menu → **Lock**).

### 3) Other ways to “template”

* **Block themes (FSE)**: `templates/*.html` and `parts/*.html` are site templates in block markup. You can use locked wrapper blocks inside to constrain editing.
* **Patterns**: reusable section blueprints; can include locked blocks to prevent accidental structure changes.

---

## Mini real project: “Team Profiles” post type

### Goal

Every **Team Profile** starts with:

* A metadata block
* An intro paragraph
* A 3-column “Team Members” container with three predefined “Team Member” items
  …and authors can only change text/images (no structure changes).

### 1) Register the post type with a predefined template (PHP)

```php
<?php
// plugin or mu-plugin file: team-profiles.php
add_action( 'init', function () {
    register_post_type( 'team_profile', [
        'label'           => 'Team Profiles',
        'public'          => true,
        'show_in_rest'    => true,
        'supports'        => [ 'title', 'editor', 'thumbnail' ],
        // Post type editor template (initial block tree)
        'template'        => [
            [ 'blocks-course/metadata-block' ],
            [ 'core/paragraph', [ 'content' => 'Write a short intro…' ] ],
            // container with 3 inner team-member blocks
            [ 'blocks-course/team-members', [ 'columns' => 3 ], [
                [ 'blocks-course/team-member' ],
                [ 'blocks-course/team-member' ],
                [ 'blocks-course/team-member' ],
            ]],
        ],
        // Editor-level locking for this post type
        'template_lock'   => 'insert', // prevent inserting extra blocks, allow moving/removing
    ] );
} );
```

> This mirrors the structure in your screenshot: a container **`blocks-course/team-members`** with three **`team-member`** children.

### 2) Make the container block strictly “content-only” (JS)

Inside the container block’s `edit` function, lock inner layout fully but let authors edit content:

```js
// blocks-course/team-members/edit.js
import { InnerBlocks } from '@wordpress/block-editor';

const TEMPLATE = [
  ['blocks-course/team-member'],
  ['blocks-course/team-member'],
  ['blocks-course/team-member'],
];

export default function Edit() {
  return (
    <div className="team-members">
      <InnerBlocks
        template={ TEMPLATE }
        templateLock="contentOnly"   // users can edit fields/media, not layout
        allowedBlocks={ [ 'blocks-course/team-member' ] }
      />
    </div>
  );
}
```

### 3) Optionally pin down each `team-member`

If each item has fixed sub-structure (e.g., image + name + role), define its own `InnerBlocks` with `templateLock="all"`:

```js
// blocks-course/team-member/edit.js
import { InnerBlocks } from '@wordpress/block-editor';

const ITEM_TEMPLATE = [
  [ 'core/image' ],
  [ 'core/heading', { level: 4, placeholder: 'Name' } ],
  [ 'core/paragraph', { placeholder: 'Role / Short bio' } ],
];

export default function Edit() {
  return (
    <div className="team-member">
      <InnerBlocks template={ ITEM_TEMPLATE } templateLock="all" />
    </div>
  );
}
```

### 4) Practical policies you can enforce

* **Authors can’t add a 4th member** (post type `template_lock: insert` and container `allowedBlocks`).
* **Authors can’t break item layout** (item has `templateLock="all"`).
* **Brand stays tight** while text/media remains editable.

---

## Advanced patterns & tips

* **Mutating core “post” type**
  If you don’t own the CPT, you can still do:

  ```php
  add_action( 'init', function () {
      $pto = get_post_type_object( 'post' );
      if ( $pto ) {
          $pto->template = [ /* … */ ];
          $pto->template_lock = 'insert';
      }
  } );
  ```

* **Conditional templates**
  Use a filter like `use_block_editor_for_post_type` or inject templates dynamically based on user role/category. For more control, enqueue an editor script that switches templates depending on context and sets `InnerBlocks` props conditionally.

* **Restrict what can be inserted globally**
  `allowed_block_types_all` filter can whitelist blocks per post type or per post.

* **Updating existing posts**
  Templates don’t retro-apply. Provide an “Insert pattern” button or a small helper that injects missing sections via a custom sidebar control if you need to migrate old content.

* **Performance/stability**
  Register custom blocks **before** the editor loads (on `init`). If a block in your template fails to register, the editor will show placeholders or strip it.

---

## Common pitfalls (and fixes)

* **Template not appearing** → Ensure CPT has `show_in_rest => true`; clear caches; verify you’re creating a **new** post; confirm array shape `[ name, attrs, inner ]`.
* **Users still rearrange blocks** → You probably used `template_lock => 'insert'` (allows move/remove). Use `'all'` at the post type level and/or `templateLock="all"` where needed.
* **Custom block missing** → The block slug must match exactly (e.g., `'blocks-course/team-member'`) and be registered on `init` (PHP) + editor scripts enqueued.
* **Need content-only editing** → Use `templateLock="contentOnly"` on the `InnerBlocks` **container** that wraps the section you want to protect.

---

## Quick cheat-sheet (copy/paste)

**Post type template (PHP)**

```php
register_post_type( 'my_cpt', [
  'show_in_rest'  => true,
  'template'      => [
    [ 'core/heading',   { level: 2, content: 'Title' } ],
    [ 'core/paragraph', { placeholder: 'Intro…' } ],
    [ 'core/columns', {}, [
      [ 'core/column', {}, [ [ 'core/image' ] ] ],
      [ 'core/column', {}, [ [ 'core/paragraph' ] ] ],
    ]],
  ],
  'template_lock' => 'insert', // false | 'insert' | 'all'
] );
```

**InnerBlocks with locking (JS)**

```js
<InnerBlocks
  template={ [
    [ 'core/image' ],
    [ 'core/heading', { level: 3, placeholder: 'Heading' } ],
  ] }
  templateLock="all"        // false | 'insert' | 'all' | 'contentOnly'
  allowedBlocks={ [ 'core/image', 'core/heading' ] }
/>
```

**Lock a specific block (attribute)**

```json
// in block markup or JSON attributes
{ "lock": { "move": false, "remove": false } }
```

**Mutate existing post type**

```php
add_action( 'init', function () {
  if ( $pto = get_post_type_object( 'post' ) ) {
    $pto->template = [ [ 'core/paragraph', [ 'content' => 'Start here…' ] ] ];
    $pto->template_lock = 'all';
  }
} );
```