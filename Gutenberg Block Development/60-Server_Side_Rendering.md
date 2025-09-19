# Server-Side Rendering in Gutenberg (`@wordpress/server-side-render`)

### Simple Explanation: What it is (and when to use it)

`@wordpress/server-side-render` is a tool that lets WordPress do the “heavy lifting” on the **server side** and then shows the result inside the block editor.

- Imagine you have a block that depends on PHP logic (like pulling posts, showing WooCommerce products, or reusing existing theme functions).
- Instead of rewriting all that logic in JavaScript, this component calls WordPress through a REST API endpoint and asks it:
  **“Hey, please render this block for me using your PHP function.”**
- WordPress runs your **`render_callback`** in PHP, creates the HTML, and sends it back to the editor so you can preview it.

### When to Use It

- ✅ When your block **must use PHP logic** that already exists.
- ✅ When your block needs **server data** but no proper REST endpoint is available.
- 🚫 Don’t use it for new blocks if you can fetch data with REST APIs and render with React — that’s faster and more modern.

### Quick Interview Phrase

You could say:

> “`@wordpress/server-side-render` lets me preview a dynamic block in the editor by asking the server to generate the HTML through the block’s PHP `render_callback`. I’d use it if the block depends on PHP logic or server data without a REST API. But for modern blocks, I’d try to render client-side first and only fall back to server-side rendering if necessary.”

---

### How it works (in plain words)

1. **In the editor**
   When you write your block’s `edit()` function, instead of writing custom JSX, you can just drop in:

   ```jsx
   <ServerSideRender block="ns/block" attributes={attributes} />
   ```

   That tells WordPress: “Please render this block preview for me.”

2. **REST API request**
   Behind the scenes, WordPress sends the block name + attributes to a REST API endpoint:
   `/wp/v2/block-renderer/ns%2Fblock`
   (It even adds the current post ID if needed.)

3. **PHP takes over**
   WordPress then calls your block’s **`render_callback`** in PHP. This is the function you wrote when registering your dynamic block. It creates the final HTML string.

4. **Back to the editor**
   That HTML is sent back through the REST API and shown instantly inside the block editor as a preview.

### Key reminder

- A **dynamic block** doesn’t save finished HTML into the post. Instead, it always asks PHP to render the final markup when needed.
- If you send attributes from JS, you also need to **declare them in PHP** — otherwise, they won’t reach your `render_callback`.

👉 **Interview one-liner:**

> “ServerSideRender works by sending the block’s name and attributes from the editor to WordPress’s REST API, which calls the block’s PHP `render_callback`. The PHP returns HTML, and the editor shows it as a live preview. That’s why it’s used for dynamic blocks.”

![ServerSideRendering Flow](<JS Libraries/ServeSideRendering.png>)

---

## API surface you’ll actually use

### 1. The Component

You usually write:

```jsx
<ServerSideRender block="ns/block" attributes={attributes} />
```

- **block** → the block’s name (`namespace/block`).
- **attributes** → the data you want to send to PHP.
- **httpMethod** → GET or POST (defaults to GET).
- **urlQueryArgs** → extra query params.
- **skipBlockSupportAttributes** → tells WordPress not to send style-related stuff if you don’t need it.
- **Placeholders** → you can customize what shows while loading, on error, or when empty (`LoadingResponsePlaceholder`, `ErrorResponsePlaceholder`, `EmptyResponsePlaceholder`).

### 2. The Hook

If you prefer hooks:

```js
const { content, status, error } = useServerSideRender({
  block: "ns/block",
  attributes,
});
```

- Returns the **content** (HTML), a **status** (loading/success/error), and **error** (if failed).
- It has built-in **debouncing**, so it won’t spam requests while the user types.

### 3. The REST Endpoint

- Endpoint: **`/wp/v2/block-renderer/<name>`**
- Works with **GET** or **POST**.
- Params:

  - `name` → block name
  - `attributes` → block data
  - `post_id` → current post ID (auto-included)
  - `context` → viewing/editing

✅ **Interview one-liner:**

> “With ServerSideRender I can either use the component or the `useServerSideRender` hook. Both send the block’s name and attributes to the `/wp/v2/block-renderer` endpoint, which runs my PHP `render_callback` and returns HTML. I can customize loading/error states, choose GET or POST, and in WP 6.2+ even skip style attributes if my PHP doesn’t need them.”

---

## Previewing dynamic blocks in the editor (patterns)

### 1. Display-only preview

- If you just want to **show what the block will look like**, use `<ServerSideRender/>`.
- The HTML is generated on the server and displayed inside the editor.
- Because it’s just HTML, you can’t click or edit it directly — so put your **controls in the sidebar** (InspectorControls).

### 2. Interactive preview

- If you want the block to **act interactive in the editor** (buttons, inputs, live updates), it’s better to build the preview with React in your `edit()` function.
- Fetch data from REST APIs or WordPress data stores, render it client-side.
- Use SSR only for the front-end output, or mix it with the new **Interactivity API** to “hydrate” server HTML with client behaviors.

### 3. Performance & UX

- When you use the **hook (`useServerSideRender`)**, you get extras:

  - It **debounces** requests so typing doesn’t flood the server.
  - You can show a **loading skeleton** while waiting.
  - You can catch and display **errors** gracefully.

✅ **Interview one-liner:**

> “For dynamic blocks, I can either use `<ServerSideRender>` to show an accurate but read-only preview, or build an interactive preview in React with REST/data stores. If I need server HTML plus interactivity, I can hydrate it with the Interactivity API. Using the hook improves UX with debouncing, loading skeletons, and error handling.”

---

## Gotchas & best practices

### 1. Declare attributes in PHP

If you send attributes from the editor, you must also **register them in PHP** with `register_block_type`. Otherwise, your `render_callback` won’t receive them.

### 2. SSR is read-only in the editor

The HTML returned by SSR is **not editable**. That’s why you place your **form fields and options in the Inspector sidebar**, not directly in the preview.

### 3. Don’t overuse SSR

Core docs call SSR a **fallback or legacy tool**.
👉 Best practice: for new blocks, use REST endpoints + client rendering for better speed and user experience. Use SSR only if you _must_ reuse PHP logic.

### 4. Security

Your `render_callback` runs for every preview request.

- Always **sanitize input** (attributes).
- Add **capability checks** if the data is sensitive.
- Never echo raw, unsafe data.

### 5. Performance

- Cache heavy queries with **transients** to avoid slowing down previews.
- Use `skipBlockSupportAttributes` if you don’t need styling attributes — smaller, faster requests.
- The hook has built-in **debounce**, so typing doesn’t flood your server.

### 6. Debugging

You can test a block’s output directly by hitting:

```
/wp-json/wp/v2/block-renderer/ns%2Fblock?attributes[...]
```

Check if the response is what you expect. Remember: the **post_id** can affect output, so keep that in mind.

✅ **Interview one-liner:**

> “With SSR, I always declare attributes in PHP, keep controls in the sidebar, sanitize inputs, and cache expensive queries. I treat it as a fallback, since modern client-rendered blocks with REST data are usually better for performance and UX.”

---

## Quick cheat-sheet

### When to Use

- ✅ Use **ServerSideRender** when you need to reuse PHP logic (`render_callback`) or don’t have a REST API.
- 🚫 Avoid it for new, interactive blocks — client rendering is faster and more flexible.

### Editor Preview

- **Display snapshot:** `<ServerSideRender … />`
- **Hook style:** `useServerSideRender()` gives `{ content, status, error }`
- Can show **loading skeletons** and **error states**.
- Put controls in the **sidebar**, since preview HTML is read-only.

### REST Endpoint

- **`/wp/v2/block-renderer/:block`** → accepts GET or POST.
- Must **declare attributes in PHP** so they reach `render_callback`.

### Props to Remember

- `block` → block name
- `attributes` → data passed to PHP
- `httpMethod`, `urlQueryArgs`
- `skipBlockSupportAttributes` (trim styling attributes if not needed)
- Placeholders → Loading / Error / Empty

### Example Use

- Great for **WooCommerce product listings, category trees, or heavy server-side logic**.
- For **interactive filters (e.g., live product search)** → better to render client-side.

✅ **Interview one-liner:**

> “ServerSideRender is best when I must reuse PHP `render_callback` logic or fetch server-only data. I use `<ServerSideRender>` or `useServerSideRender` for editor previews, always declare attributes in PHP, keep controls in the sidebar, and prefer client rendering for new, interactive blocks.”

---

## Interview Quetions

**Q1. What is `@wordpress/server-side-render` and when should I use it?**
**A.** It’s a way to show a **dynamic block preview** in the editor by letting the server generate the HTML. Use it if you already have PHP logic or no REST API. For new blocks, client-side rendering is usually better.

**Q2. How do you preview a dynamic block in the editor?**
**A.** Use `<ServerSideRender/>` or `useServerSideRender()`. They fetch the HTML for you. Remember: the preview is **read-only**, so put inputs in the sidebar.

**Q3. What must be registered server-side for SSR to work?**
**A.** Your block must have a **PHP `render_callback`**, and any attributes you send must be **declared in PHP**, or else they won’t show up.

**Q4. GET or POST? What about payload size?**
**A.** Both work. By default it uses **GET**, but if you send lots of data, it can switch to **POST**.

**Q5. What changed recently that’s useful?**
**A.** WordPress 6.2 added `skipBlockSupportAttributes` to skip style attributes in the request. Also, the new `useServerSideRender` hook makes it easier to handle loading and errors.

**Q6. Why do WooCommerce/product blocks sometimes use SSR?**
**A.** Because product/category lists already have heavy PHP queries. SSR reuses that code. But for interactive features like filters or search, client-side is better.

**Q7. Trade-offs vs client rendering?**
**A.** SSR matches front-end output exactly, but is slower and not interactive. Client rendering is faster and feels better in the editor, but you may need to build extra APIs.

**Q8. Performance & security tips?**
**A.** Cache heavy queries, sanitize all inputs, check user permissions, and don’t send extra attributes you don’t need.

---

### **Interview one-liner:**

> “`@wordpress/server-side-render` lets me preview a dynamic block in the editor by asking the server to generate the HTML through the block’s PHP `render_callback`. I’d use it if the block depends on PHP logic or server data without a REST API. But for modern blocks, I’d try to render client-side first and only fall back to server-side rendering if necessary.”

> “ServerSideRender works by sending the block’s name and attributes from the editor to WordPress’s REST API, which calls the block’s PHP `render_callback`. The PHP returns HTML, and the editor shows it as a live preview. That’s why it’s used for dynamic blocks.”

> “With ServerSideRender I can either use the component or the `useServerSideRender` hook. Both send the block’s name and attributes to the `/wp/v2/block-renderer` endpoint, which runs my PHP `render_callback` and returns HTML. I can customize loading/error states, choose GET or POST, and in WP 6.2+ even skip style attributes if my PHP doesn’t need them.”

> “For dynamic blocks, I can either use `<ServerSideRender>` to show an accurate but read-only preview, or build an interactive preview in React with REST/data stores. If I need server HTML plus interactivity, I can hydrate it with the Interactivity API. Using the hook improves UX with debouncing, loading skeletons, and error handling.”

> “With SSR, I always declare attributes in PHP, keep controls in the sidebar, sanitize inputs, and cache expensive queries. I treat it as a fallback, since modern client-rendered blocks with REST data are usually better for performance and UX.”

> “ServerSideRender is best when I must reuse PHP `render_callback` logic or fetch server-only data. I use `<ServerSideRender>` or `useServerSideRender` for editor previews, always declare attributes in PHP, keep controls in the sidebar, and prefer client rendering for new, interactive blocks.”
