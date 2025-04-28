# Gutenberg Block Development with React: Deep Dive into MediaPlaceholder 🎯

When building custom Gutenberg blocks, one thing is clear: **media handling** is essential. Whether you're creating a blog card, a hero section, a testimonial block, or a gallery — you’ll often need to **let users upload or select media** inside the editor itself.

This is where **`MediaPlaceholder`** from `@wordpress/block-editor` comes in — a **lifesaver** for WordPress developers!

In this deep dive, you'll learn:

- What `MediaPlaceholder` is 🧩
- Why and when you should use it 🎯
- Key props and concepts to master ⚙️
- Practical use cases with real examples 🛠️

Let’s jump in!

---

## 📌 What Is `MediaPlaceholder`?

`MediaPlaceholder` is a **React component** provided by WordPress' Gutenberg editor.  
> Its purpose is to **display a media selection UI** (upload or select from the media library) **inside the block editor**, *before* the actual media (image, video, file, etc.) is available ❗️

👉 In simple words: **It creates a beautiful UI for media uploading or selection before the user has picked something**.

> Once the media is selected/uploaded, you replace the `MediaPlaceholder` with the actual media preview ❗️

✅ It supports:
- Uploading new files ❗️
- Choosing from the existing media library ❗️
- Drag-and-drop uploads ❗️
- Multiple media types (images, videos, audio, files) ❗️

---

## 🧠 Core Concepts of `MediaPlaceholder`

### 1. **Importing MediaPlaceholder**
```javascript
import { MediaPlaceholder } from '@wordpress/block-editor';
```
You need to import it from `@wordpress/block-editor`, not from `@wordpress/components`.

---

### 2. **Basic Structure**
The typical usage looks like this:

```javascript
<MediaPlaceholder
    icon="format-image"
    labels={{ title: 'Upload Image' }}
    onSelect={(media) => console.log(media)}
    accept="image/*"
    allowedTypes={['image']}
    multiple={false}
/>
```

- **icon**: WordPress dashicon name or a custom SVG.
- **labels**: Titles or instructions you want to show.
- **onSelect**: The callback when a media file is selected/uploaded.
- **accept**: Mime types (optional).
- **allowedTypes**: Array of WordPress media types (image, video, etc.).
- **multiple**: Boolean to allow selecting multiple media items.

---

### 3. **Handling Selected Media**

> Usually, once a media is selected, you **update your block attributes** with media details like `url`, `id`, `alt`, etc ❗️❗️❗️

Example:

```javascript
onSelect={(media) => setAttributes({ 
    mediaURL: media.url, 
    mediaID: media.id 
})}
```

---

### 4. **Conditionally Showing MediaPlaceholder**

> Typically, you show `MediaPlaceholder` **only when no media has been selected** yet ❗️❗️❗️:

```javascript
{ !mediaID ? (
    <MediaPlaceholder ... />
) : (
    <img src={mediaURL} alt="Selected media" />
) }
```

---

## 🔥 Practical Deep Dive: Full Example

Let's create a **custom Gutenberg block** that lets users upload an image with `MediaPlaceholder`.

```javascript
import { registerBlockType } from '@wordpress/blocks';
import { MediaPlaceholder } from '@wordpress/block-editor';
import { Button } from '@wordpress/components';
import { useState } from '@wordpress/element';

registerBlockType('myplugin/simple-image', {
    title: 'Simple Image Upload',
    icon: 'format-image',
    category: 'media',
    attributes: {
        mediaID: { type: 'number' },
        mediaURL: { type: 'string' },
    },
    edit: ({ attributes, setAttributes }) => {
        const { mediaID, mediaURL } = attributes;

        const onSelectMedia = (media) => {
            setAttributes({
                mediaID: media.id,
                mediaURL: media.url,
            });
        };

        return (
            <div className="simple-image-block">
                { !mediaID ? (
                    <MediaPlaceholder
                        icon="format-image"
                        labels={{ title: 'Upload an Image' }}
                        onSelect={onSelectMedia}
                        accept="image/*"
                        allowedTypes={['image']}
                        multiple={false}
                    />
                ) : (
                    <div>
                        <img src={mediaURL} alt="Uploaded" style={{ width: '100%' }} />
                        <Button 
                            onClick={() => setAttributes({ mediaID: null, mediaURL: null })} 
                            isSecondary
                        >
                            Replace Image
                        </Button>
                    </div>
                )}
            </div>
        );
    },
    save: ({ attributes }) => {
        const { mediaURL } = attributes;
        return mediaURL ? <img src={mediaURL} alt="Uploaded" /> : null;
    }
});
```

✅ This block lets the user upload an image or select from the media library.  
✅ Once uploaded, it shows the image + a button to replace it.

---

## 💡 Real-World Use Cases for `MediaPlaceholder`

| Use Case                          | Description |
|:-----------------------------------|:------------|
| **Image or Video Upload**          | Allow users to upload or choose an image/video inside a hero block, testimonial, etc. |
| **Custom Gallery Blocks**          | Let users pick multiple images to build a custom gallery. |
| **Audio Uploaders**                | For podcast, music, or radio blocks where users upload audio files. |
| **Document Upload (PDFs, etc.)**   | Let users upload and embed PDFs, Word docs, etc. inside blocks. |
| **Interactive Media Cards**        | Enable background media (image/video) uploads for interactive blocks.|

---

## ⚙️ Advanced Customizations

- **Customize allowedTypes**  
  You can restrict the user to only certain file types (`['audio', 'video']`, etc.).
  
- **Custom Instructions**  
  Pass your own instruction text in the `labels` prop to guide users clearly.
  
- **Handle Multiple Selections**  
  Set `multiple={true}` to allow uploading/selecting several files at once.

- **Drag-and-Drop UX**  
  `MediaPlaceholder` automatically supports drag-and-drop file uploads.

---

# 🚀 Final Thoughts

The **MediaPlaceholder** component is a **critical tool** in building media-driven blocks efficiently inside Gutenberg.  
It makes your user interface **intuitive, WordPress-native**, and **much faster** to develop.

Whenever you need a **pre-upload state** for media — whether it’s images, videos, audio, or documents — think **MediaPlaceholder**!

---
