## 📌 `wp_list_pages()`


The `wp_list_pages()` function in WordPress is used to **generate a list of WordPress pages as HTML output**. Retrieves or displays a list of pages hierarchically (or hierarchical post type items) in **list (li)** format. It's a useful tool for developers and theme designers when they want to display a menu or a list of links to all the pages on a site without needing to manually create the list.

**Example:**
```php
wp_list_pages(array(
    "title_li" => NULL,
    "child_of" => $findChildrenOf,
    "sort_column" => "menu_order"
));
```

### 1. **Purpose of `wp_list_pages()`**

**The primary purpose of `wp_list_pages()` is to retrieve and display a hierarchical list of all published pages. This function automatically formats the pages into an unordered list (`<ul>`), which can then be styled using CSS to suit your website’s needs.**

It’s often used in themes to create page navigation or sitemaps, and it’s a more automated way to handle page links than manually coding each page URL.

### 2. **Core Concepts**

- **Pages vs. Posts:** In WordPress, pages are different from posts. Pages are static pieces of content (like "About," "Contact," or "Services"), while posts are dynamic and often organized by categories or tags.
- **Hierarchical Structure:** Pages in WordPress can have a hierarchical structure, meaning a page can have sub-pages (child pages). The `wp_list_pages()` function respects this hierarchy and displays the pages accordingly.

### 3. **Syntax of `wp_list_pages()`**

Here is the basic syntax for using `wp_list_pages()`:

```php
<?php wp_list_pages( $args ); ?>
```

- `$args` is an array or a query string of arguments that modify the behavior of the function.

### 4. **Common Arguments and Parameters**

The `wp_list_pages()` function accepts several arguments that let you control how the pages are displayed. Here are some of the most commonly used ones:

| Argument              | Description                                                                                                   | Example                                      |
|-----------------------|---------------------------------------------------------------------------------------------------------------|----------------------------------------------|
| `title_li`            | Adds a title for the list of pages. If set to an empty string, the `<li>` title will be hidden.                | `title_li='My Pages'`                        |
| `sort_column`         | Specifies how the pages should be ordered. Common values: `post_title`, `menu_order`, `post_date`.            | `sort_column='post_title'`                   |
| `exclude`             | A comma-separated list of page IDs to exclude from the list.                                                   | `exclude='4,7,9'`                            |
| `include`             | A comma-separated list of page IDs to include in the list, excluding all others.                              | `include='2,3,5'`                            |
| `child_of`            | Displays only the child pages of a specific page by specifying its ID.                                         | `child_of=2`                                 |
| `depth`               | Controls how many levels in the hierarchy are to be included. `0` for all levels; `1` for top-level only.     | `depth=1`                                    |
| `link_before`/`link_after` | Adds text or HTML before or after each page link.                                                        | `link_before='<span>'` `link_after='</span>'`|

### 5. **Use Cases of `wp_list_pages()`**

#### 5.1. **Creating a Simple Page Menu**

The simplest use of `wp_list_pages()` to display a list of all pages in a theme’s sidebar might look like this:

```php
<?php wp_list_pages( array( 'title_li' => '' ) ); ?>
```

This will generate an unordered list (`<ul>`) with each page in the website linked in a list item (`<li>`).

#### 5.2. **Displaying Child Pages Only**

If you want to display only the child pages of a specific page, you can use the `child_of` parameter. For example:

```php
<?php wp_list_pages( array( 'child_of' => 42, 'title_li' => '' ) ); ?>
```

This will display only the pages that are children of the page with the ID of `42`.

#### 5.3. **Excluding Certain Pages**

To exclude specific pages from being listed, use the `exclude` parameter:

```php
<?php wp_list_pages( array( 'exclude' => '4,9,23', 'title_li' => '' ) ); ?>
```

This will exclude pages with the IDs `4`, `9`, and `23` from the list.

#### 5.4. **Limiting the Depth of the Page Tree**

You can control how many levels of pages are shown in the list using the `depth` parameter:

```php
<?php wp_list_pages( array( 'depth' => 1, 'title_li' => '' ) ); ?>
```

This example will only display the top-level pages (and no child pages).

### 6. **Practical Implementation**

A common scenario where `wp_list_pages()` is used is in the creation of a site map. Here is an example of a site map code:

```php
<h2>Site Map</h2>
<ul>
    <?php wp_list_pages( array( 'title_li' => '', 'sort_column' => 'menu_order, post_title' ) ); ?>
</ul>
```

This code displays an organized list of pages in menu order, which is useful for a hierarchical navigation structure.

### 7. **Alternatives to `wp_list_pages()`**

While `wp_list_pages()` is powerful, many modern WordPress themes and developers now use **custom menus** created in the WordPress admin area (`Appearance > Menus`) for more flexibility. You can then display these menus using the `wp_nav_menu()` function, which provides more control over styling and menu behavior.

### 8. **Best Practices**

- **Use `wp_nav_menu()` if possible:** For more dynamic and customizable menu structures, consider using `wp_nav_menu()`.
- **Avoid hard-coding:** Avoid hard-coding page links directly into your template files; using `wp_list_pages()` makes your site more dynamic and easier to manage.

### Summary

`wp_list_pages()` is a powerful function for generating a list of pages in a WordPress site. It is ideal for creating navigation menus, sitemaps, or any list of pages. While it's versatile and easy to use, `wp_nav_menu()` is often preferred in modern themes for better flexibility and customization.