# What is `withNotices` in WordPress?

`withNotices` is a **Higher-Order Component (HOC)** provided by the WordPress `@wordpress/components` package. It allows your component (like a custom block or sidebar panel) to **programmatically display notices** — such as success messages, warnings, or errors — in the Gutenberg interface.

It’s mainly used in **plugin-side UI interactions**, like form validation, API feedback, and user actions (e.g., uploading images, saving settings, etc.).

---

## 🎯 Purpose of `withNotices`

The core **purpose** of `withNotices` is to **inject notice management props** into your component so that you can:

- **Display notices** (info, success, warning, error)
- **Clear notices** manually
- Provide users with immediate feedback inside the Gutenberg editor

Think of it like:
🔔 “Hey, something just happened — let’s tell the user right inside the editor.”

---

## 🧠 Core Concepts

### 1. **Higher-Order Component (HOC)**

`withNotices` wraps your component and passes `noticeOperations` and `noticeUI` as props.

```js
import { withNotices } from "@wordpress/components";
```

You use it like this:

```js
export default withNotices(MyComponent);
```

Now your `MyComponent` receives:

- `noticeOperations`: functions to create/clear notices
- `noticeUI`: the actual rendered notice UI (optional to display inline)

---

### 2. **Key Props Provided**

#### 🛠️ `noticeOperations`

This object provides:

- `createNotice( type, message, options )`
  To create a notice (`type`: 'success' | 'info' | 'warning' | 'error')

- `removeNotice( id )`
  To remove a specific notice

#### 🖼️ `noticeUI`

A pre-rendered `Notice` component (not always used — optional)

---

## 🧪 Real-World Use Cases

### ✅ 1. **Show Success Message After an API Call**

```js
import { isBlobURL } from "@wordpress/blob";
import { Spinner, withNotices } from "@wordpress/components";

function Edit({ attributes, setAttributes, noticeUI, noticeOperations }) {
  const { url, alt } = attributes;
  const onUploadError = (message) => {
    noticeOperations.removeAllNotices();
    noticeOperations.createErrorNotice(message);
  };

  return (
    <div {...useBlockProps()}>
      {url && (
        <div
          className={`wp-block-blocks-course-team-member-img${
            isBlobURL(url) ? " is-loading" : ""
          }`}
        >
          <img src={url} alt={alt} />
          {isBlobURL(url) && <Spinner />}
        </div>
      )}
      <MediaPlaceholder
        icon={"admin-users"}
        onSelect={onSelectImage}
        onSelectURL={onSelectURL}
        onError={onUploadError}
        accept="image/*"
        allowedTypes={["image"]}
        disableMediaButtons={url}
        notices={noticeUI}
      />
    </div>
  );
}

export default withNotices(Edit);
```

---

### ❌ 2. **Handle Image Upload Errors**

```js
const MyImageUpload = ({ noticeOperations }) => {
  const onUploadError = (errorMessage) => {
    noticeOperations.createNotice("error", errorMessage, {
      isDismissible: true,
    });
  };

  return (
    <MediaUpload
      onError={onUploadError}
      render={({ open }) => <Button onClick={open}>Upload</Button>}
    />
  );
};

export default withNotices(MyImageUpload);
```

---

### 👁️ 3. **Render Notice Inline with `noticeUI`**

This allows you to **visually embed** the notice directly in your component's UI.

---

## 🧩 Where to Use `withNotices`

- Custom block settings panels
- Plugin sidebar components
- MediaUpload handlers
- REST API interaction feedback
- Forms inside the editor (e.g., contact forms, plugin options)

---

## 📝 Notes & Best Practices

- You **must handle cleanup** if notices should be removed after actions.
- Don’t use it to spam users with messages — show **only meaningful feedback**.
- Combine it with async/await or `try/catch` for maximum clarity.
- Works well with **InspectorControls**, **Sidebar plugins**, and **Meta Boxes**.

---

## 🧰 Alternative: `useDispatch( 'core/notices' )` (Modern Hook)

If you're using functional components with hooks, you can **avoid `withNotices`** and use the **`useDispatch`** hook:

```js
import { useDispatch } from "@wordpress/data";

const { createNotice } = useDispatch("core/notices");

createNotice("success", "This is a success!");
```

But `withNotices` is **still useful** when you're working with **class components** or **wrapper UI elements**.

---

## 📦 Import Path

```js
import { withNotices } from "@wordpress/components";
```

Or in package form:

```bash
npm install @wordpress/components
```

---

## 🧭 Final Thoughts

`withNotices` is your Gutenberg-friendly way to give users **real-time feedback** — crucial for a polished user experience.

Use it when:

- You need to display **confirmation, warnings, or errors**
- You're working inside Gutenberg’s React environment
- You want to hook into Gutenberg’s native **notice system** visually and contextually
