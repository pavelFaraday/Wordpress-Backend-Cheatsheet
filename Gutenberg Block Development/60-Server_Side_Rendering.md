# Server-Side Rendering in Gutenberg (`@wordpress/server-side-render`) — Deep Dive + Interview Prep

## What it is (and when to use it)

**`@wordpress/server-side-render`** is a React component + hook that asks WordPress to render your **dynamic block** on the server and returns the HTML to preview **inside the editor**. It calls the REST endpoint **`/wp/v2/block-renderer/:block`**, which in turn runs your block’s PHP `render_callback`. It’s ideal when your block **must** reuse existing PHP logic or tightly couples to server data that doesn’t have suitable REST endpoints. New blocks are encouraged to render client-side and use REST/Store APIs instead; treat SSR as a **fallback/legacy mechanism**. ([WordPress Developer Resources][1])

---

## How it works (editor preview → PHP → HTML)

1. In `edit()`, render `<ServerSideRender block="ns/block" attributes={…} />` **or** use the `useServerSideRender` hook.
2. The component sends the block name + attributes to **`/wp/v2/block-renderer/ns%2Fblock`** (GET or POST). The current post ID is automatically included when relevant. ([WordPress Developer Resources][1])
3. WordPress invokes your block’s **`render_callback`** and returns the HTML string, which the editor displays. If you pass attributes, **you must also declare them in PHP** when registering the block so they reach your callback. ([WordPress Developer Resources][1])

> **Dynamic block refresher:** a “dynamic” block outputs markup on request via `render_callback` (vs. static blocks that save HTML in post content). ([WordPress Developer Resources][2])

---

## API surface you’ll actually use

* **Component:** `<ServerSideRender … />`
  Props you’ll care about:

  * `block` (string), `attributes` (object), optional `httpMethod` (`'GET'|'POST'`, default GET), `urlQueryArgs` (object), **`skipBlockSupportAttributes`** (bool), and placeholders: `LoadingResponsePlaceholder`, `ErrorResponsePlaceholder`, `EmptyResponsePlaceholder`. ([WordPress Developer Resources][1])
* **Hook:** `useServerSideRender({ block, attributes, httpMethod, urlQueryArgs, skipBlockSupportAttributes })`
  Gives `{ content, status, error }`, with built-in **debouncing** to avoid chatty requests while you type. ([WordPress Developer Resources][1])
* **Endpoint:** `POST /wp/v2/block-renderer/<name>` (also supports GET); args include `name`, `attributes`, optional `post_id`, `context`. ([WordPress Developer Resources][3])
* **6.2+ prop:** `skipBlockSupportAttributes` lets you **exclude block-support styling attributes** from the request payload if your PHP callback doesn’t need them. ([Make WordPress][4])

---

## Previewing dynamic blocks in the editor (patterns)

* **Display-only preview:** Use `<ServerSideRender/>` for an accurate server HTML snapshot; wire **controls in the sidebar** (since the preview HTML itself is read-only). ([Ollie WordPress Block Theme][5])
* **Interactive preview:** Prefer a **client-rendered** `edit()` using data stores/REST endpoints; reserve SSR for front-end output only. If you need server-computed HTML + client interactivity, consider the **Interactivity API** to hydrate server output. ([WordPress Developer Resources][6])
* **Performance UX:** With the hook, show a skeleton while `status==='loading'`, handle `error`, and debounce comes for free. ([WordPress Developer Resources][1])

---

## WooCommerce & product-related blocks (what’s common)

* **Why SSR shows up:** Product/category listings can be **expensive** or depend on WooCommerce PHP templates/settings. For example, WooCommerce changed the **Product Categories List** block to SSR to avoid shuttling big payloads through the editor. ([The WooCommerce Developer Blog][7])
* **Where not to use SSR:** Many newer WooCommerce blocks (e.g., **Product Collection**) emphasize client-side interactivity and optimized fetching; they render in JS and talk to Woo endpoints/stores for filtering, pagination, etc. SSR here would undercut responsiveness. ([The WooCommerce Developer Blog][8])
* **Guideline:** Use SSR for **read-only previews** of server-templated output (menus, taxonomy trees, receipt/order snippets), or when **legacy PHP** must be reused. Use client rendering for **filterable, live-updating** product UIs.

---

## Gotchas & best practices

* **Declare attributes on the server** (in `register_block_type`) if you send them via SSR; otherwise they won’t reach `render_callback`. ([WordPress Developer Resources][1])
* **SSR is read-only HTML** in the canvas; put inputs in the **Inspector (sidebar) controls**. ([Ollie WordPress Block Theme][5])
* **Don’t overuse it:** Core docs explicitly position SSR as **fallback/legacy**—build new blocks around endpoints/client rendering for the best UX. ([WordPress Developer Resources][1])
* **Security:** Sanitize attributes, enforce capability checks in your callback, and avoid echoing raw data. (The renderer runs on every preview.)
* **Performance:** Cache expensive queries (transients), and consider `skipBlockSupportAttributes` to trim request size. The hook’s debounce helps avoid hammering your server. ([Make WordPress][4])
* **Debugging:** Hit `/wp-json/wp/v2/block-renderer/ns%2Fblock` with attributes to inspect the raw response; remember **`post_id`** can affect output. ([WordPress Developer Resources][3])

---

## Quick cheat-sheet

* **Use when:** You must reuse **PHP `render_callback`** logic or lack APIs; need accurate server HTML inside the editor. **Avoid** for brand-new, interactive blocks. ([WordPress Developer Resources][1])
* **Editor preview:** `<ServerSideRender/>` **or** `useServerSideRender()`; show skeletons/errors; keep controls in sidebar. ([WordPress Developer Resources][1])
* **Endpoint:** `/wp/v2/block-renderer/:block` (GET/POST). **Attributes must be declared in PHP**. ([WordPress Developer Resources][3])
* **Props to remember:** `block`, `attributes`, `httpMethod`, `urlQueryArgs`, `skipBlockSupportAttributes`, plus `Loading/Error/Empty` placeholders. ([WordPress Developer Resources][1])
* **WooCommerce:** Good for **server-heavy listings/trees**; use client rendering for **filterable product collections**. ([The WooCommerce Developer Blog][7])

---

## Practical use cases

* **Archives/Category lists** (WP core example), **Product category trees** (WooCommerce), **dynamic pricing tables**, **latest-X widgets** that must mirror PHP templates or respect server-side settings/caps. ([WordPress Developer Resources][1])
* **Transactional snippets** (order summaries/receipts) where correctness and permissions trump client-side speed.
* **Legacy migrations**: when rewriting everything to client render isn’t feasible, SSR lets you bridge.

---

## Interview-style talking points (with strong answers)

**Q1. What is `@wordpress/server-side-render` and when should I use it?**
**A.** It’s a component/hook that previews a **dynamic block** by calling `/wp/v2/block-renderer/:block`, which runs the block’s **PHP `render_callback`** and returns HTML. Core positions it as a **fallback/legacy** path—prefer client-side rendering with REST/Stores for new features. Use SSR when you must reuse heavy PHP or lack suitable endpoints. ([WordPress Developer Resources][1])

**Q2. How do you preview a dynamic block in the editor?**
**A.** Use `<ServerSideRender/>` or `useServerSideRender()`. They handle fetching, loading/error states, and debouncing. Keep UI controls in the **Inspector** because the preview is **read-only HTML**. ([WordPress Developer Resources][1], [Ollie WordPress Block Theme][5])

**Q3. What must be registered server-side for SSR to work?**
**A.** The block must be registered with a **`render_callback`**, and any attributes you send must be **declared in PHP**; otherwise they won’t be passed to your callback. ([WordPress Developer Resources][1])

**Q4. GET or POST? What about payload size?**
**A.** Both are supported. The API defaults to GET but supports POST for bigger payloads; the hook includes logic for this and debounces requests. ([WordPress Developer Resources][1])

**Q5. What changed recently that’s useful?**
**A.** **`skipBlockSupportAttributes`** (WP 6.2) to exclude block-support style attributes from the SSR request, reducing payload/complexity. There’s also a dedicated **`useServerSideRender`** hook for fine-grained loading/error UI. ([Make WordPress][4], [WordPress Developer Resources][1])

**Q6. Why do WooCommerce/product blocks sometimes use SSR?**
**A.** For taxonomy/product listings where server templates and complex queries are already implemented (e.g., **Product Categories List** moved to SSR for efficiency). But modern product browsing (e.g., **Product Collection**) favors client interactivity via data stores and endpoints. ([The WooCommerce Developer Blog][7])

**Q7. Trade-offs vs client rendering?**
**A.** SSR guarantees parity with front-end PHP but yields a **non-interactive** preview and extra network trips; client rendering offers better editor UX and responsiveness but requires endpoints and duplication unless you architect carefully. Core recommends client rendering for new blocks. ([WordPress Developer Resources][1])

**Q8. Performance & security tips?**
**A.** Cache expensive queries in `render_callback`; sanitize attributes; check capabilities; and keep the attributes payload lean (use `skipBlockSupportAttributes`). ([Make WordPress][4])

