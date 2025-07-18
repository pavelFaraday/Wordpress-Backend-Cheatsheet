# From HTML to Blocks: How WordPress Parses and Renders Post Content

## 🧠 Overview: How WordPress Parses and Renders Post Content in the Editor

There are **three main stages**:

1. **Editor Loads HTML From Database**
2. **HTML is Parsed into an Array of Blocks**
3. **Blocks Are Used to Render the Visual Editor**

Let’s explore each part with details and code.

---

### 🗃 1. HTML Is Fetched from the Database

When editing a post in Gutenberg, WordPress fetches **raw HTML block content** from the database. This is what’s stored in the `post_content` field.

You can get this content with:

```js
const html = wp.data.select("core/editor").getEditedPostContent();
```

This gives you block-based HTML like:

```html
<!-- wp:paragraph -->
<p>Hello World</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":12,"sizeSlug":"large"} -->
<figure>
  <img src="image.jpg" />
  <figcaption>Caption</figcaption>
</figure>
<!-- /wp:image -->
```

This HTML is the **only thing** stored in the DB — no arrays, no JSON. Just this HTML-with-comments.

---

### 🔍 2. HTML is Parsed into a Block Array

To make this HTML **usable**, Gutenberg parses it into structured block data using:

```js
const blocks = wp.blocks.parse(html);
```

This returns an **array of block objects**:

```js
[
  {
    name: 'core/paragraph',
    attributes: { content: 'Hello World' },
    innerBlocks: [],
    innerHTML: '<p>Hello World</p>',
    ...
  },
  {
    name: 'core/image',
    attributes: {
      id: 12,
      sizeSlug: 'large',
      url: 'image.jpg',
      caption: 'Caption',
      ...
    },
    ...
  }
]
```

This array is called the **Post State**.

It is also available via:

```js
wp.data.select("core/block-editor").getBlocks();
```

This Post State array drives the **visual block editor UI**.

---

### 🧩 How the `parse()` Function Works

`wp.blocks.parse()` figures out block structure by using **HTML comments** called **delimiters**.

For example:

```html
<!-- wp:paragraph -->
<p>Hi there</p>
<!-- /wp:paragraph -->
```

The `<!-- wp:... -->` comment contains **metadata**, like:

- The block name (`core/paragraph`)
- Optional inline JSON with attributes (`{"id":12}`)

It uses this information to:

- Identify the **block type**
- Extract **attributes** (like `url`, `alt`, etc.)
- Store **inner content** or child blocks

---

### 🧠 Where Do Attributes Come From?

Block attributes are extracted in two ways:

#### 1. **From the Delimiter Comment** (`<!-- wp:image { ... } -->`)

- For attributes **not visible** in the HTML
- Example: `id`, `sizeSlug`

#### 2. **From HTML Content** (using `selector` and `source`)

- For visible attributes
- Example: `src` from `<img>`, caption from `<figcaption>`

Each block defines how its attributes are parsed in its `attributes` definition, like:

```js
attributes: {
  url: {
    type: 'string',
    source: 'attribute',
    selector: 'img',
    attribute: 'src'
  },
  caption: {
    type: 'string',
    source: 'html',
    selector: 'figcaption'
  }
}
```

So during parsing, WordPress reads from both:

- The **comment object**
- The **HTML structure**

---

### ✍️ Editing Content in the Visual Editor

When you add or edit blocks in the editor, it modifies the **Post State array** in the store (`core/block-editor`).

You can get it again via:

```js
wp.data.select("core/block-editor").getBlocks();
```

---

### 🔁 Serializing Back to HTML

After editing, the block array is **serialized back to HTML** using:

```js
const html = wp.blocks.serialize(blockArray);
```

This produces the new block-based HTML to be saved into the database.

Example:

```html
<!-- wp:quote -->
<blockquote><p>This is a quote.</p></blockquote>
<!-- /wp:quote -->
```

And that becomes the new `post_content`.

---

## 🔄 The Full Cycle

Here's the full loop of post content in Gutenberg:

| Step | Description                                                   |
| ---- | ------------------------------------------------------------- |
| 1️⃣   | Load saved HTML from DB using `getEditedPostContent()`        |
| 2️⃣   | Parse HTML into block array using `wp.blocks.parse()`         |
| 3️⃣   | Store block array in Redux via `wp.data`                      |
| 4️⃣   | Visual editor renders based on block array                    |
| 5️⃣   | User edits content, updates Redux store                       |
| 6️⃣   | Block array is serialized to HTML via `wp.blocks.serialize()` |
| 7️⃣   | HTML is saved back to DB (`post_content`)                     |

![The Full Cycle](/Gutenberg%20Block%20Development/JS%20Libraries/Post-Parsing-Cycle.png)

---

## 🧱 Why This Matters for Developers

- You can **access post content as blocks** or HTML depending on your use case.
- You can **dynamically manipulate blocks**, insert or remove blocks, or modify attributes.
- Understanding parsing/serialization lets you build **custom blocks** and even full block-based apps.

---

## 📚 Bonus: Useful APIs

### Access block data

```js
wp.data.select("core/block-editor").getBlocks();
```

### Modify post content

```js
wp.data.dispatch("core/editor").editPost({ content: newHTML });
```

### Parse HTML to blocks

```js
wp.blocks.parse(html);
```

### Serialize blocks to HTML

```js
wp.blocks.serialize(blockArray);
```

---

## ✅ Conclusion

WordPress Gutenberg treats `post_content` as **HTML with block comments**, but under the hood it’s transformed into a structured **block array** using `wp.blocks.parse()`. This array is what powers the visual editing experience — and can be read or modified using `wp.data`.

If you're building blocks or tools for the editor, understanding this flow is **critical**.
