# Namespaces in PHP (Generally) & Wordpress Theme Development

## What are PHP namespaces

Namespaces are a way to group code under a unique path so your classes, functions, and constants do not collide with others. Think of them like folders for symbols.

![Namespaces](Slides/namespaces1.png)
![Namespaces](Slides/namespaces2.png)

---

```php
<?php
namespace MyTheme\Utils;

class Url {}
function is_external() {}
const VERSION = '1.0.0';
```

- Fully qualified name: `\MyTheme\Utils\Url`
- Inside `MyTheme\Utils`, `new Url()` refers to `\MyTheme\Utils\Url`
- Outside, you must import or fully qualify:

  ```php
  use MyTheme\Utils\Url;
  $u = new Url();
  // or
  $u = new \MyTheme\Utils\Url();
  ```

# Why namespaces matter in WordPress themes

- Avoid collisions with plugins and core which live in the global namespace
- Organize theme logic by domain: Services, Assets, Blocks, REST, Customizer, Admin
- Enable PSR-4 autoloading and clean OOP without giant function prefixes
- Make testable modules that are easy to move between projects

# Core concepts you must know

## 1) Declaring and importing

- `namespace` must be the first statement in a file
- One namespace per file is the usual practice
- Import with `use` and optional aliasing

```php
<?php
namespace MyTheme\Blocks;

use MyTheme\Support\Assets;
use Vendor\Lib\Client as ApiClient;
```

Group imports:

```php
use MyTheme\{Support\Assets, Support\Dates};
```

## 2) Name resolution rules

- Classes: no fallback - `new Service()` looks in current namespace only, use `\Service` or an import to reach global
- Functions: PHP checks current namespace, then global
- Constants: no fallback - use full or imported names

## 3) Paths, sub-namespaces, and PSR-4

- Convention: namespace mirrors folders
- PSR-4 mapping in `composer.json`:

  ```json
  {
    "autoload": {
      "psr-4": {
        "MyTheme\\": "src/"
      }
    }
  }
  ```

- Include Composer’s autoloader in `functions.php`:

  ```php
  require get_template_directory() . '/vendor/autoload.php';
  ```

## 4) Namespaced callbacks for hooks

Hook names are strings, not namespaced, but your callbacks can be.

```php
<?php
namespace MyTheme;

function setup() {
  add_theme_support('post-thumbnails');
}

add_action('after_setup_theme', __NAMESPACE__ . '\\setup');
```

You can also use static methods or invokable classes:

```php
add_action('init', [\MyTheme\Assets\Manager::class, 'register']);
```

## 5) Each template file has its own namespace

Files you include can declare their own namespace. If not declared, they are in the global namespace.

```php
// functions.php
namespace MyTheme;
// ...

// src/Blocks/Register.php
namespace MyTheme\Blocks;
// ...
```

Template parts loaded with `get_template_part()` are just PHP files. They can be namespaced, but most teams keep template files global and keep logic in `src/`.

## 6) Interop with core functions and template tags

Core functions like `wp_enqueue_script`, `get_the_title` live in the global namespace.

- Call them with a leading backslash or import nothing and call normally:

  ```php
  \wp_enqueue_script('my-handle', $url, [], null, true);
  // calling wp_enqueue_script() also works because functions fall back to global
  ```

- For clarity and a tiny performance win, many devs prefix with `\` inside namespaced code

# Typical use cases in practice

1. Theme bootstrap

- Group setup logic inside `MyTheme\Setup\Theme::boot`
- Register supports, menus, image sizes, editor settings

2. Asset management

- Centralize enqueue logic in `MyTheme\Assets\Manager`
- Add helpers for versioning, RTL, async, and preload

3. Gutenberg blocks

- `Register::register_blocks` for server registered blocks
- Namespaced render callbacks avoid global function clutter

4. Customizer or Settings pages

- `MyTheme\Admin\Customizer` with sections and controls
- `MyTheme\Admin\SettingsPage` for options with capability checks

5. Integration modules

- `MyTheme\Integrations\ACF`, `\Woo`, `\WPML` to isolate plugin specific code

6. Utilities and domain services

- `MyTheme\Support\Sanitizer`, `Strings`, `Images`, `Urls`

# Gotchas and best practices

- Do not mix multiple namespaces in one file unless you really must
- Place `namespace ...;` as the first statement
- Avoid global helper functions - prefer static class methods or dedicated utility classes
- Use `__NAMESPACE__ . '\\function'` when registering callbacks from the same file
- Prefer `ClassName::class` over hard coded strings
- Keep template files mostly presentational - put logic in namespaced classes
- Be careful with `require` paths - use WordPress helpers like `get_template_directory()`
- Avoid side effects on file load - register hooks in a boot method

# Quick cheat-sheet

**Declare**

```php
namespace Vendor\Package;
```

**Import**

```php
use Vendor\Package\Service;
use Vendor\Package\Utils as U;
```

**Call core functions from a namespace**

```php
\add_action('init', [Service::class, 'init']);
\esc_html($str);
```

**Register hook with namespaced function**

```php
namespace MyTheme;

function setup() { /* ... */ }
\add_action('after_setup_theme', __NAMESPACE__ . '\\setup');
```

**Render callback for a dynamic block**

```php
\register_block_type($dir, ['render_callback' => [\MyTheme\Blocks\Register::class, 'render_hero']]);
```

**Composer autoload**

```json
"autoload": { "psr-4": { "MyTheme\\": "src/" } }
```

```php
require __DIR__ . '/vendor/autoload.php';
```

---

# Interview Topics

## 🧠 1. What problem do namespaces solve in PHP?

**Model answer:**
Namespaces prevent name collisions between classes, functions, or constants that share the same name but come from different packages or parts of the project. In WordPress, this is especially important because core and plugins all share the global namespace. Without namespaces, two `Helper` or `Utils` classes could overwrite each other, causing fatal errors. Namespaces allow me to group code logically (like `MyTheme\Blocks\Hero`) and keep my theme’s scope isolated from everything else.

## 🧠 2. How do you declare and use a namespace in PHP?

**Model answer:**
You declare a namespace at the top of the file using:

```php
namespace MyTheme\Blocks;
```

Then you can use its elements with `use` statements or full paths:

```php
use MyTheme\Support\Assets;
$assets = new Assets();
```

If I need a global function like `wp_enqueue_script`, I prefix it with a backslash (`\wp_enqueue_script()`), because WordPress functions live in the global namespace.

## 🧠 3. What happens if you call a global WordPress function inside a namespace without the backslash?

**Model answer:**
PHP first looks for that function inside the current namespace. If it doesn’t exist, it falls back to the global namespace for functions only (not classes or constants). So calling `wp_enqueue_script()` usually still works, but to be explicit and avoid any confusion, I prefer `\wp_enqueue_script()`. It’s a small performance improvement and keeps intent clear.

## 🧠 4. Can you explain the difference between fully qualified, qualified, and unqualified names?

**Model answer:**

- **Fully qualified name** - Starts with `\`, e.g., `\MyTheme\Assets\Manager`, and is resolved from the global namespace.
- **Qualified name** - Includes namespace but no leading `\`, e.g., `Assets\Manager`, which PHP resolves relative to the current namespace.
- **Unqualified name** - Only the class name, like `Manager`, which PHP looks for inside the current namespace.
  Knowing these rules avoids autoloading errors or class not found exceptions.

## 🧠 5. How do you use namespaces with WordPress hooks like `add_action()` or `add_filter()`?

**Model answer:**
Hooks are global, but your callback can be namespaced.
Example:

```php
namespace MyTheme\Setup;

function init_theme() {
  add_theme_support('post-thumbnails');
}

add_action('after_setup_theme', __NAMESPACE__ . '\\init_theme');
```

`__NAMESPACE__ . '\\function'` ensures the callback string resolves correctly.
For classes:

```php
add_action('init', [\MyTheme\Assets\Manager::class, 'register']);
```

This keeps the global space clean while making hook references explicit.

## 🧠 6. How do namespaces integrate with Composer and PSR-4 autoloading in WordPress themes?

**Model answer:**
Composer’s PSR-4 autoload maps a namespace to a directory. Example in `composer.json`:

```json
"autoload": { "psr-4": { "MyTheme\\": "src/" } }
```

Then I run `composer dump-autoload` and require `vendor/autoload.php` in `functions.php`.
Now any class under `MyTheme\...` is autoloaded automatically without `require` calls.
This is the modern way to structure large WordPress themes or plugins.

## 🧠 7. How do namespaces improve maintainability and scalability in WordPress development?

**Model answer:**
Namespaces enforce a modular architecture. Each domain - like `Setup`, `Assets`, `Blocks`, or `Admin` - can live in its own namespace and evolve independently.
It’s easier to read, test, and reuse. For example, I can move `MyTheme\Assets\Manager` into another project without breaking anything.
It also helps IDEs provide better autocompletion and refactoring tools.
In large client projects, this prevents “function pollution” that older themes suffer from.

## 🧠 8. Can you use multiple namespaces in one PHP file? Should you?

**Model answer:**
Yes, you technically can:

```php
namespace A { /* ... */ }
namespace B { /* ... */ }
```

But it’s **not recommended** in real-world WordPress development because it complicates readability and autoloading.
Best practice: one namespace per file that matches your directory structure. This aligns with PSR-4 and keeps your architecture predictable.

## 🧠 9. How do you call a namespaced function or class from a different namespace?

**Model answer:**
You either use the **fully qualified name** or import it with `use`:

```php
use MyTheme\Assets\Manager;
Manager::register();
```

or:

```php
\MyTheme\Assets\Manager::register();
```

`use` makes the code cleaner, while the full path avoids ambiguity when only used once.
When calling between modules in WordPress (like `Blocks` using a helper in `Support`), I prefer `use` imports for clarity.

## 🧠 10. How do namespaces affect testing and reusability of WordPress components?

**Model answer:**
With namespaces, I can test isolated classes without loading the entire WordPress environment. For example, `MyTheme\Support\Sanitizer` can be unit tested directly.
Because everything is modular and independent, I can extract these classes into standalone Composer packages or reuse them in other projects easily.
In legacy WordPress code, global functions make this impossible. Namespacing brings professional-grade code organization and long-term maintainability.

## ⚡ Bonus Short “Rapid-Fire” Questions

| Question                                              | Quick Answer                                            |
| ----------------------------------------------------- | ------------------------------------------------------- |
| What keyword returns the current namespace?           | `__NAMESPACE__`                                         |
| How do you reference a class name as a string safely? | `ClassName::class`                                      |
| Can you import functions or constants?                | Yes, since PHP 5.6 using `use function` or `use const`. |
| What’s the namespace separator?                       | Backslash (`\`)                                         |
| Why not put everything in the global namespace?       | It risks collisions with core or plugins.               |

---

## 📋 Cheat-Note for Interview Flashcards

| Concept             | Example                                             |
| ------------------- | --------------------------------------------------- |
| Declare             | `namespace MyTheme\Assets;`                         |
| Import              | `use MyTheme\Support\Sanitizer;`                    |
| Global function     | `\wp_enqueue_script()`                              |
| Namespaced callback | `add_action('init', __NAMESPACE__ . '\\register');` |
| Class callback      | `[MyClass::class, 'method']`                        |
| Autoload            | `"psr-4": { "MyTheme\\": "src/" }`                  |
| Root include        | `require __DIR__ . '/vendor/autoload.php';`         |
