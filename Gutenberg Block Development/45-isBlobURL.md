# 🔍 `isBlobURL` in WordPress Gutenberg – A Deep Dive

## 🧠 What is `isBlobURL`?

In Gutenberg block development, `isBlobURL` is a **utility function** provided by the `@wordpress/blob` package. It is used to determine whether a given URL is a [Blob URL](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL) — **a special type of URL that points to data stored in the browser's memory.**

### 📦 Source: `@wordpress/blob`

To use it in your code, import it like this:

```js
import { isBlobURL } from "@wordpress/blob";
```

---

## 🧱 Blob URLs: A Quick Refresher

Blob URLs are <u>**temporary**</u> URLs that reference data stored as `Blob` objects in memory. **They are typically used when uploading files (like images or videos) before those files are permanently stored on the server**.

They look like this:

```
blob:http://localhost:3000/79f2c437-c999-4bc7-a6b7-798af01016c4
```

---

## 🎯 Purpose of `isBlobURL`

The main purpose of `isBlobURL` is to **detect whether a media URL is still a temporary blob**, as opposed to a permanent media URL from the WordPress Media Library.

This is useful when:

- Uploading media via the `MediaUpload` or `MediaPlaceholder` components
- Rendering temporary previews of images or videos before they’re saved
- Preventing attempts to persist or reference blob URLs as final content
- Handling cleanup after upload is completed

---

## 🔩 Core Concept

```js
function isBlobURL( url: string ): boolean;
```

It returns `true` if the URL starts with the `blob:` scheme.

Internally, it's just checking the URL string with `startsWith("blob:")`.

---

## Real Example

```js
import { useEffect, useState } from "@wordpress/element";
import { isBlobURL } from "@wordpress/blob";
import { Spinner } from "@wordpress/components";

export default function Edit({ attributes, setAttributes }) {
  const { url, alt } = attributes;
  const [blobURL, setBlobURL] = useState();

  const onSelectImage = (image) => {
    if (!image || !image.url) {
      setAttributes({ url: undefined, id: undefined, alt: "" });
      return;
    }
    setAttributes({ url: image.url, id: image.id, alt: image.alt });
  };

  useEffect(() => {
    if (!id && isBlobURL(url)) {
      // If the URL is a blob URL, we need to clear the ID and alt text.
      setAttributes({ url: undefined, alt: "" });
    }
  }, []);
  useEffect(() => {
    if (isBlobURL(url)) {
      // If the URL is a blob URL, we need to create a blob URL.
      setBlobURL(url);
    } else {
      revokeBlobURL(blobURL);
      setBlobURL(undefined);
    }
  }, [url]);

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
        onSelectURL={() => {}}
        onError={() => {}}
        accept="image/*"
        allowedTypes={["image"]}
        disableMediaButtons={url}
      />
    </div>
  );
}
```

### Example Explanation

#### 🧼 What is `revokeBlobURL(url)`?

**Function:**

`revokeBlobURL` tells the browser to **release the memory used by a blob URL**.

When you no longer need a blob URL (e.g., the image has been uploaded to the server), **you should clean it up**, otherwise it could cause memory leaks.

### ✅ In your code:

```js
useEffect(() => {
  if (isBlobURL(url)) {
    setBlobURL(url); // Save it in local state
  } else {
    revokeBlobURL(blobURL); // Clean up if not needed anymore
    setBlobURL(undefined);
  }
}, [url]);
```

- If the image is still a blob → store it.
- If it’s **no longer** a blob → **release the old blob** from memory.

#### 🧠 useState: What’s it doing?

```js
const [blobURL, setBlobURL] = useState();
```

This is used to **keep track of the current blob URL** in React’s state.

Why?

Because when a new image is selected, the component might re-render — you don’t want to lose track of the current blob you need to revoke later. So you're storing it in state.

#### 🪝 useEffect 1: Cleanup on Mount

```js
useEffect(() => {
  if (!id && isBlobURL(url)) {
    setAttributes({ url: undefined, alt: "" });
  }
}, []);
```

- This runs only **once** when the component mounts.
- If the current image is a blob URL **without an ID**, you clean it immediately.
- Why? A blob URL with no associated media ID means the media was likely removed or never uploaded.

🧠 Smart default guard to avoid broken states.

#### 🪝 useEffect 2: React to URL changes

```js
useEffect(() => {
  if (isBlobURL(url)) {
    setBlobURL(url);
  } else {
    revokeBlobURL(blobURL);
    setBlobURL(undefined);
  }
}, [url]);
```

This runs every time `url` changes.

- If the new URL is a blob → save it.
- If it’s a **server URL** → revoke any previous blob URL and clean state.

🔥 This ensures you **don’t keep any old blob in memory** and that UI reflects the state accurately.

#### 💡 Summary

| Concept         | Purpose                                                           |
| --------------- | ----------------------------------------------------------------- |
| `isBlobURL`     | Checks if a URL is a temporary browser blob (used before upload). |
| `revokeBlobURL` | Frees browser memory when blob is no longer needed.               |
| `useState`      | Tracks the current blob URL to safely revoke it later.            |
| `useEffect` (1) | Clears invalid blob URLs on initial mount.                        |
| `useEffect` (2) | Reacts to `url` changes — stores blob or revokes old ones.        |

---

#### ✅ Why This Matters

In Gutenberg, managing blob URLs properly:

- Improves performance (by avoiding memory leaks),
- Prevents broken image previews,
- Ensures clean transitions from **upload → saved media**,
- Keeps your block editor responsive and bug-free.

---

## 🛠 Practical Use Cases in Gutenberg

### 1. **Previewing Uploaded Images**

When users upload an image but haven’t saved the post yet, the image is rendered using a blob URL.

```js
{
  isBlobURL(image.url) && <Spinner />;
}
<img src={image.url} alt={image.alt} />;
```

If the image is still a blob (not saved to media library), you might:

- Show a spinner
- Disable save
- Show a placeholder or visual cue

---

### 2. **Prevent Saving Unsaved Blobs**

To avoid saving unsaved image blobs to post content:

```js
if (isBlobURL(image.url)) {
  return; // wait until it's uploaded and saved
}
```

This ensures that your block content remains stable and doesn't reference ephemeral blob URLs.

---

### 3. **Clean Up After Upload**

When using `createBlobURL()` to preview file uploads, it's best practice to revoke the Blob URL once done.

```js
useEffect(() => {
  return () => {
    if (isBlobURL(image.url)) {
      revokeBlobURL(image.url);
    }
  };
}, [image.url]);
```

This prevents memory leaks by releasing the in-memory Blob when it’s no longer needed.

---

## ⚙️ Sample Scenario: Image Upload Block

Here’s a real-world example:

```js
import { MediaUpload, MediaPlaceholder } from "@wordpress/block-editor";
import { isBlobURL, revokeBlobURL } from "@wordpress/blob";
import { useEffect, useState } from "@wordpress/element";

const MyImageBlock = () => {
  const [image, setImage] = useState({});

  useEffect(() => {
    return () => {
      if (image?.url && isBlobURL(image.url)) {
        revokeBlobURL(image.url);
      }
    };
  }, [image?.url]);

  return (
    <div>
      {!image?.url ? (
        <MediaPlaceholder
          onSelect={(media) => {
            setImage(media);
          }}
          allowedTypes={["image"]}
          multiple={false}
          labels={{ title: "Upload an image" }}
        />
      ) : (
        <>
          {isBlobURL(image.url) && <Spinner />}
          <img src={image.url} alt={image.alt || ""} />
        </>
      )}
    </div>
  );
};
```

---

## ✅ Summary: When to Use `isBlobURL`

| Use Case                            | Why Use `isBlobURL`                               |
| ----------------------------------- | ------------------------------------------------- |
| During media uploads                | Detect if the file is not yet uploaded            |
| Rendering temporary image previews  | Show spinners or placeholders for blob URLs       |
| Avoiding blob URLs in saved content | Prevent unstable or broken links                  |
| Cleaning up memory                  | Revoke blob URLs after use with `revokeBlobURL()` |

---

## 📌 Final Thoughts

`isBlobURL` may look simple, but it's crucial when working with media uploads in Gutenberg. Without it, you risk:

- Saving broken image URLs
- Creating memory leaks
- Confusing UX during uploads

It helps you build robust and professional media-handling experiences.
