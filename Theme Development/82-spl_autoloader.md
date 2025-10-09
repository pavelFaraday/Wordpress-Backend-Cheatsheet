# `spl_autoload_register` in php & WP

## What is `spl_autoload_register`?

`spl_autoload_register` tells PHP how to **find and include class files on demand**. When your code first uses `new MyClass()`, PHP asks your autoloader: “where is the file for this class?”. Your autoloader maps the class name to a file path and `require`s it. No more dozens of `require_once` lines.

Why it exists:

- Avoid manual `require` spaghetti
- Faster dev and fewer bugs from missing includes
- Enables modern architecture with namespaces and folders

## How autoloading works in 3 steps

1. PHP sees a **class/interface/trait** name it does not know.
2. It runs your registered autoloader callbacks **in order**.
3. The first autoloader that can map the name to a file **includes** it.

Key terms:

- **Autoload stack:** multiple autoloaders can be registered. Order matters.
- **PSR-4 mapping:** standard that maps namespaces to directories.
- **Classmap:** array that maps class names to exact files.
- **Composer autoloader:** a generated autoloader that supports PSR-4, classmap, and files.

![`spl_autoloader()`](Slides/spl_autoloader.png)
![`spl_autoloader()`](Slides/spl_autoloader2.png)

---

## Core function signature

```php
spl_autoload_register(
    callable $autoload,
    bool $throw = true,
    bool $prepend = false
);
```

- `$autoload`: your function that receives the class name.
- `$throw`: throw if registration fails.
- `$prepend`: put your autoloader at the **front** of the stack.

Helpers:

```php
spl_autoload_functions();  // list autoloaders
spl_autoload_unregister($callable); // remove one
```

Deprecated:

- `__autoload()` is deprecated. Use SPL autoloaders instead.

---

## Practical WP use cases

1. **Plugin architecture** - split code by domain:

```
includes/src/
  Admin/Settings.php
  Admin/Menu.php
  REST/CubesController.php
  Model/Cube.php
  Support/Assets.php
```

Each file holds one class. Autoload maps namespace to folders.

2. **MU-plugins or custom platform**
   Register a global autoloader in `mu-plugins/autoload.php` and add multiple prefixes:

```php
$map = [
  'Company\\Shared\\' => WPMU_PLUGIN_DIR . '/company-shared/src/',
  'Company\\Store\\'  => WP_CONTENT_DIR . '/plugins/company-store/includes/src/',
];

spl_autoload_register(function($class) use ($map) {
    foreach ($map as $prefix => $dir) {
        if (strncmp($prefix, $class, strlen($prefix)) === 0) {
            $relative = substr($class, strlen($prefix));
            $file = $dir . str_replace('\\','/',$relative) . '.php';
            if (is_readable($file)) { require $file; }
            return;
        }
    }
});
```

3. **Legacy theme without Composer**
   Add a small autoloader in `functions.php` for your theme namespace:

```php
add_action('after_setup_theme', function () {
    require get_theme_file_path('inc/autoload.php');
});
```

4. **Classmap for non-standard names**
   When file names do not match class names, keep a manual map:

```php
$classmap = [
  'Pik\\PixelsLand\\Legacy\\Pdf' => __DIR__.'/lib/pdf/legacy_pdf.php',
];
spl_autoload_register(function($class) use ($classmap) {
    if (isset($classmap[$class])) require $classmap[$class];
});
```

---

## Best practices in WordPress

- **Use namespaces** with a unique vendor prefix to avoid collisions: `Pik\PixelsLand\...`
- **One class per file**. Match namespace path to folders.
- **Prefer Composer autoload** in modern plugins and themes.
- **Guard your namespace** in the autoloader to avoid doing unnecessary work.
- **Keep autoloader fast** - minimal I/O, avoid `file_exists` loops, prefer direct path calculation.
- **Cache wins** - OPcache reduces include cost. Autoloading is safe with OPcache.
- **Do not rely on `spl_autoload_extensions`**. File suffixes should be `.php`.
- **Play nice with other autoloaders** - never `die()` in an autoloader, just return if not yours.
- **Check before using** - `class_exists(FQCN)` is fine and triggers autoload.

Common pitfalls:

- Wrong base dir or prefix string off by one.
- Case sensitivity on some filesystems.
- Mixing slashes or forgetting `.php`.
- Registering the autoloader too late. Do it early in plugin bootstrap.

---

## Quick cheat-sheet

- Register: `spl_autoload_register(function($class){ /* map + require */ });`
- Multiple autoloaders allowed. Order matters. Use `$prepend=true` to run first.
- Scope to your namespace: check prefix, then map to file.
- PSR-4 rule: `Vendor\Package\Sub\Class -> baseDir/Sub/Class.php`
- Composer: define `"autoload": {"psr-4": {"Vendor\\Package\\": "src/"}}`
- WordPress plugin: require `vendor/autoload.php` in main plugin file.
- Prefer namespaces + PSR-4 over custom conventions.
- Use `class_exists()` to lazy-load optional features.
- Do not use deprecated `__autoload()`.

---

## Short summary

Autoloading loads class files automatically when first used. `spl_autoload_register` lets you define the mapping rule from class name to file path. In WordPress, combine namespaces + PSR-4 + Composer for clean structure and fewer includes. Keep the autoloader small, fast, and scoped to your prefix.

---

## Interview-style talking points with strong model answers

**1) What problem does autoloading solve?**
It removes manual `require` lines and ensures the correct file is included exactly when the class is needed. This reduces boilerplate, avoids duplicate includes, and keeps architecture modular.

**2) How does `spl_autoload_register` work with multiple autoloaders?**
It manages a stack of callbacks. PHP calls them in order until one includes the right file. You can prepend yours if it must run first. Use `spl_autoload_functions()` to debug the stack.

**3) Explain PSR-4 quickly.**
PSR-4 maps a namespace prefix to a base directory. Replace backslashes with slashes and append `.php`. Example: `Pik\PixelsLand\Admin\Menu` maps to `includes/src/Admin/Menu.php`.

**4) Why prefer Composer in WP?**
Composer generates a robust autoloader supporting PSR-4, classmaps, and optimization. It saves effort, improves performance with optimized classmaps, and avoids custom bugs.

**5) How do you keep an autoloader fast?**
Scope to a known prefix, compute direct paths, avoid directory scanning, and let OPcache do its job. In Composer, run `composer dump-autoload -o` to optimize classmaps.

**6) How do you avoid conflicts in WP?**
Use a globally unique namespace like `Vendor\Project\...`. Never declare global classes with common names. Do not intercept classes from other plugins.

**7) What is the difference between PSR-0, PSR-4, and classmap?**
PSR-0 is legacy. PSR-4 is the current standard with a simple namespace-to-path rule. Classmap maps specific class names to specific files - good for legacy or non-standard layouts.

**8) How do you handle legacy code that does not follow PSR-4?**
Add a small classmap for the odd parts or place an adapter layer. Gradually refactor to PSR-4 while keeping the autoloader mapping stable.

**9) When would you use `$prepend=true`?**
When your autoloader must run before others, for example a platform autoloader that resolves core classes first.

**10) How do you integrate autoloading into a WP plugin lifecycle?**
Require your autoloader in the main plugin file upfront, then instantiate classes on `plugins_loaded` or later. From there, class usage triggers autoload automatically.

---

# 10 interview questions with strong model answers

**Q1. What does `spl_autoload_register` do and when is it called?**
**A.** It registers a callback that maps class names to files. PHP calls it automatically the first time an unknown class/interface/trait is referenced.

**Q2. Show a tiny PSR-4 autoloader and explain the mapping.**
**A.** See the earlier 10-line function: check prefix, strip it, replace backslashes with slashes, append `.php`, `require` if readable. `Vendor\Pkg\Foo\Bar` -> `base/Foo/Bar.php`.

**Q3. How do you combine Composer autoload with a custom fallback in WP?**
**A.** Try `vendor/autoload.php` first. If not present in production or on shared hosting, fall back to a light custom PSR-4 autoloader. This keeps the plugin portable.

**Q4. Does autoloading work for interfaces and traits?**
**A.** Yes. The same mechanism applies. If PHP needs an interface or a trait and it is unknown, it triggers autoload.

**Q5. How to debug autoload problems in WP?**
**A.** Check `spl_autoload_functions()` order, verify prefix and base path, confirm case-sensitive file names, log the requested class name, and test with `class_exists('FQCN', true)`.

**Q6. Why is `__autoload()` discouraged?**
**A.** It only allowed one autoloader and is deprecated. SPL autoloaders allow multiple callbacks with better control and diagnostics.

**Q7. When would you choose a classmap instead of PSR-4?**
**A.** When dealing with legacy file names that do not match class names or for performance tuning in huge codebases where Composer’s optimized classmap can be faster.

**Q8. Is autoloading slower than manual `require`?**
**A.** In practice, it is at least as fast and often faster because only needed classes are loaded, OPcache caches compiled code, and Composer can optimize lookups.

**Q9. How do you protect your autoloader from touching other vendors’ classes?**
**A.** Early exit if the class does not start with your namespace prefix. Never attempt to load unknown prefixes.

**Q10. Where do you register the autoloader in a plugin or theme?**
**A.** As early as possible in the main entry file. For plugins, that is the main plugin file. For themes, in `functions.php` or an included `inc/bootstrap.php` during theme setup.
