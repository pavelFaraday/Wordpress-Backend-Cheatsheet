# 🧱 Templates & Template Locking in Gutenberg (Simple Guide)

## 💡 What are Templates?

Templates are **ready-made layouts** built with blocks.
They control **how your posts and pages look** — like “page blueprints”.

👉 Example:
Every “Blog Post” could use the same layout - title, image, content, author bio.

---

## 🗂 Where templates live

In **block themes**:

```
/templates/single.html
/templates/page.html
/parts/header.html
/parts/footer.html
```

In **classic themes or plugins**:
You can define them in PHP when you register a post type.

---

## 🎯 Why Templates exist

Templates help you:

- ✅ Keep a **consistent design** across posts.
- ✅ Save time - no need to rebuild layout every time.
- ✅ Prevent mistakes - authors can’t delete or move key blocks.
- ✅ Control what can and can’t be edited.

👉 Think of them as **"guardrails"** for editors.

---

## ⚙️ Core Ideas

### 1️⃣ Post Type Templates (in PHP)

You can define a **default block structure** for a post type.
This layout appears **when you create a new post**.

**Example:**

```php
register_post_type( 'team_profile', [
  'show_in_rest'  => true,
  'template' => [
    [ 'core/heading', [ 'content' => 'Team Introduction' ] ],
    [ 'core/paragraph', [ 'content' => 'Write something about your team...' ] ],
    [ 'core/columns', {}, [
      [ 'core/column', {}, [ [ 'core/image' ] ] ],
      [ 'core/column', {}, [ [ 'core/paragraph', [ 'content' => 'Name' ] ] ] ],
    ] ],
  ],
  'template_lock' => 'insert',
] );
```

🟢 This means:

- Every new “Team Profile” starts with this structure.
- Users can edit text and images but can’t **add new blocks**.

---

### 2️⃣ Template Locking

Locking means **how much control editors have** inside the layout.

| Level           | Lock type     | What it does                             |
| --------------- | ------------- | ---------------------------------------- |
| `false`         | none          | Free editing                             |
| `'insert'`      | Partial lock  | Can move/delete blocks but can’t add new |
| `'all'`         | Full lock     | Can’t move, remove, or add               |
| `'contentOnly'` | Hide block UI | Can only change text or images           |

**Used in two places:**

- In PHP (`'template_lock' => 'insert'`)
- In JavaScript with `<InnerBlocks templateLock="all" />`

---

### 3️⃣ Example: “Team Profiles” CPT

**Goal:**
Each “Team Profile” post has:

- One metadata block
- One intro paragraph
- One “Team Members” container with 3 team members
- Authors can only change content, not layout

---

### 🧩 Step 1: Register the CPT

```php
register_post_type( 'team_profile', [
  'label'         => 'Team Profiles',
  'public'        => true,
  'show_in_rest'  => true,
  'template_lock' => 'insert',
  'template'      => [
    [ 'blocks-course/metadata-block' ],
    [ 'core/paragraph', [ 'content' => 'Write short intro…' ] ],
    [ 'blocks-course/team-members', [], [
      [ 'blocks-course/team-member' ],
      [ 'blocks-course/team-member' ],
      [ 'blocks-course/team-member' ],
    ] ],
  ],
] );
```

---

### 🧩 Step 2: InnerBlocks - Lock layout (JS)

```js
<InnerBlocks
  template={[
    ["blocks-course/team-member"],
    ["blocks-course/team-member"],
    ["blocks-course/team-member"],
  ]}
  templateLock="contentOnly" // only text/media editable
  allowedBlocks={["blocks-course/team-member"]}
/>
```

---

### 🧩 Step 3: Each Team Member (JS)

```js
<InnerBlocks
  template={[
    ["core/image"],
    ["core/heading", { level: 4, placeholder: "Name" }],
    ["core/paragraph", { placeholder: "Role / Bio" }],
  ]}
  templateLock="all" // full lock
/>
```

🟢 This way:

- Each profile has 3 members.
- Authors can’t delete or add extra members.
- They can only change text or photo.

---

## 🪄 Quick Tips

✅ Use `'show_in_rest' => true` - required for Gutenberg.
✅ Templates only work on **new posts**, not old ones.
✅ Register all custom blocks before the editor loads.
✅ Use `'contentOnly'` for safest content editing.
✅ Use `allowed_block_types_all` filter to limit which blocks can appear.

---

## 🧰 Common Problems

| Problem                           | Reason                 | Fix                            |
| --------------------------------- | ---------------------- | ------------------------------ |
| Template not showing              | Missing `show_in_rest` | Add `'show_in_rest' => true`   |
| Template works only on new posts  | Normal behavior        | Only affects new ones          |
| Authors can move or delete blocks | Wrong lock type        | Use `'all'` or `'contentOnly'` |
| Custom block missing              | Not registered yet     | Register it early in `init`    |

---

## ⚡ Quick Copy-Paste Cheatsheet

**PHP**

```php
register_post_type( 'my_cpt', [
  'show_in_rest'  => true,
  'template'      => [
    [ 'core/heading', { level: 2, content: 'Title' } ],
    [ 'core/paragraph', { placeholder: 'Intro…' } ],
  ],
  'template_lock' => 'insert',
] );
```

**JS**

```js
<InnerBlocks
  template={[
    ["core/image"],
    ["core/heading", { level: 3, placeholder: "Heading" }],
  ]}
  templateLock="all"
  allowedBlocks={["core/image", "core/heading"]}
/>
```

---

## 🔍 Summary

| Concept                          | What it does                               |
| -------------------------------- | ------------------------------------------ |
| **Template**                     | Starting block layout for new posts        |
| **Template parts**               | Reusable header/footer blocks              |
| **Template Lock**                | Controls what users can change             |
| **`templateLock="contentOnly"`** | Hides structure - only text/media editable |

👉 Templates = structure.
👉 Template Locking = control.
