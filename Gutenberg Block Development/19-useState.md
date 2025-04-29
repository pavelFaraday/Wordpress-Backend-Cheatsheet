# Gutenberg Block Development with React: Deep Dive into `useState` 🚀

If you're diving into **Gutenberg block development** using **React**, mastering **`useState`** is absolutely essential.  
This hook is one of the first building blocks (pun intended!) you’ll need for managing **dynamic state** inside your custom blocks.

Today, we’ll break down everything you need to know about `useState` — **purpose**, **core concepts**, and **real-world examples** inside Gutenberg block development.

---

## What is `useState`? 🎯

At its core, `useState` is a **React Hook** that **adds state** to functional components.  

In Gutenberg block development, your block’s editing interface (the **Editor UI**) often needs to react to user input:  
whether it's **typing text**, **toggling options**, or **adjusting settings**.

Since Gutenberg relies on **React functional components**, `useState` allows you to create **dynamic, interactive editing experiences** without needing to write full class components.

Here’s the basic syntax:

```javascript
const [stateVariable, setStateFunction] = useState(initialValue);
```

- `stateVariable`: The current value of the state.
- `setStateFunction`: A function you call to update the state.
- `initialValue`: The starting value for the state.

---

## Purpose of `useState` in Gutenberg Block Development 🔥

> In Gutenberg, blocks need to handle **temporary states** **while editing** in the block editor ❗️ 
These temporary states are different from **saved attributes** (which are persisted into post content) ❗️
>✅ `useState` is perfect for managing **temporary**, **non-persistent** UI state ❗️❗️❗️
It **enhances the editing experience** but doesn't directly affect the saved content.


For example:
- Handling **modal open/close** state
- Managing **input fields** before committing changes to block attributes
- Toggling **preview/edit** modes
- Managing **loading** or **validation** states


---

## Core Concepts Behind `useState` 💡

Let's go deeper into **how** and **why** you use it.

### 1. Initialization
When the component first renders, `useState` initializes the state with the **value you provide**.

```javascript
const [count, setCount] = useState(0);
```
- `count` starts at `0`.
- Calling `setCount(1)` updates `count` to `1`, **triggering a re-render**.

---

### 2. Updating State
You always use the **setter function** (`setCount`) to update the state.

You **should not** directly modify the state variable.

✅ **Correct** 👇❗️❗️❗️👇:
```javascript
setCount(count + 1);
```

❌ Incorrect:
```javascript
count = count + 1; // Won't cause re-render
```

Updating state **properly** ensures your block **re-renders** to reflect the new UI.

---

### 3. Functional Updates
If your new state depends on the previous state, use a **function inside setState**: 👇❗️❗️❗️👇

```javascript
setCount(prevCount => prevCount + 1);
```

This avoids issues when React batches updates asynchronously.

---

## Practical Use Cases of `useState` in Gutenberg Block Development 🛠️

Now let's connect it to **real Gutenberg block development** scenarios.

### 1. **Temporary Form State**

Suppose you're creating a **custom Call To Action (CTA) block** with a popup modal for editing link text.

```javascript
import { useState } from '@wordpress/element';

function Edit({ attributes, setAttributes }) {
    const [tempLinkText, setTempLinkText] = useState('');

    return (
        <div>
            <input
                type="text"
                value={tempLinkText}
                onChange={(e) => setTempLinkText(e.target.value)}
                placeholder="Enter link text"
            />
            <button onClick={() => setAttributes({ linkText: tempLinkText })}>
                Save
            </button>
        </div>
    );
}
```

- `tempLinkText` holds **temporary input**.
- When "Save" is clicked, you **commit** it to the block’s attributes.

---

### 2. **Modal Open/Close Toggle**

Imagine a **settings panel modal** inside your block:

```javascript
import { useState } from '@wordpress/element';
import { Modal, Button } from '@wordpress/components';

function Edit() {
    const [isModalOpen, setIsModalOpen] = useState(false);

    return (
        <>
            <Button isPrimary onClick={() => setIsModalOpen(true)}>
                Open Settings
            </Button>

            {isModalOpen && (
                <Modal title="Settings" onRequestClose={() => setIsModalOpen(false)}>
                    <p>Settings content here...</p>
                </Modal>
            )}
        </>
    );
}
```

- `isModalOpen` controls if the settings **modal** is shown.
- Clicking the button toggles it **open/close**.

---

### 3. **Loading State in Async Actions**

When fetching remote data inside a Gutenberg block (like external images or posts), you often need a **loading spinner**.

```javascript
const [isLoading, setIsLoading] = useState(false);

const fetchData = async () => {
    setIsLoading(true);
    const response = await fetch('https://api.example.com/posts');
    const data = await response.json();
    setIsLoading(false);
    // Set fetched data into state or attributes
};

return (
    <div>
        {isLoading ? (
            <Spinner />
        ) : (
            <Button onClick={fetchData}>Load Posts</Button>
        )}
    </div>
);
```

- The `Spinner` appears **while loading**.
- Once data is fetched, **normal UI** returns.

---

## Key Points to Remember ⚡

| Concept                        | Explanation                                                    |
|:--------------------------------|:---------------------------------------------------------------|
| `useState` adds local state     | Needed for dynamic block UI interactions (e.g., input, toggle) |
| Never mutate state directly     | Always use the setter function (like `setValue`)               |
| Functional updates are safer    | Especially if new state depends on old state                  |
| Temporary vs Persistent state   | `useState` = temporary, `attributes` = saved                   |
| Triggers re-render              | State updates cause the block to refresh visually             |

---

## Conclusion 🎉

In Gutenberg block development, **`useState` is your best friend** for creating rich, dynamic block editing experiences.

It’s lightweight, simple to use, and unlocks **dynamic behaviors** inside your block editor — whether it’s handling input forms, modals, loading states, or toggling views.

**Mastering `useState` = building better, more powerful Gutenberg blocks!**

---