## 📌 Difference between `get_pages()` and `wp_list_pages()`

In WordPress, both `get_pages()` and `wp_list_pages()` are functions used to retrieve and display pages. However, they serve slightly different purposes and are used in different scenarios.


`get_pages()` is very similar to `wp_list_pages();`. **The only difference is that `wp_list_pages();` --> will handle <u>outputting the pages onto the screen</u>, whereas `get_pages();` --> just returns the pages in memory.**

### `get_pages()`
The `get_pages()` function is used to retrieve an array of page objects from the database based on specified criteria. You can use this function to get a list of pages and then manipulate or display them as needed within your theme or plugin.

#### **Syntax**:
```php
$pages = get_pages( $args );
```

#### **Parameters**:
The `$args` parameter is an optional array of arguments to filter the results. Some common arguments include:

- `child_of` (int): Retrieve only pages that are children of the specified page ID.
- `parent` (int): Retrieve only pages with the specified parent ID.
- `sort_order` (string): Order of the pages, can be `'ASC'` (ascending) or `'DESC'` (descending).
- `sort_column` (string): Column to sort pages by, such as `'post_title'`, `'menu_order'`, `'post_date'`, etc.
- `number` (int): Number of pages to retrieve. Defaults to retrieving all pages.
- `include` (array): Array of page IDs to include.
- `exclude` (array): Array of page IDs to exclude.

#### **Example**:
```php
$pages = get_pages( array(
    'sort_column' => 'menu_order',
    'sort_order'  => 'ASC',
    'parent'      => 0
) );

foreach ( $pages as $page ) {
    echo '<h2>' . $page->post_title . '</h2>';
    echo '<p>' . $page->post_content . '</p>';
}
```

This example fetches all top-level pages (with no parent), sorts them by menu order in ascending order, and displays their title and content.

### **`wp_list_pages()`**

🔗 Reference - [wp_list_pages](25-wp_list_pages.md)

The `wp_list_pages()` function is used to generate and display a list of pages as HTML `<li>` elements within a `<ul>` container. It automatically outputs the list of pages in a hierarchical format and is most commonly used for creating page navigation menus.

#### **Syntax**:
```php
wp_list_pages( $args );
```

#### **Parameters**:
The `$args` parameter is an optional array of arguments to control the output. Some common arguments include:

- `title_li` (string): The title to display above the list of pages. If set to an empty string, no title will be shown.
- `sort_column` (string): Column to sort pages by, such as `'post_title'`, `'menu_order'`, `'post_date'`, etc.
- `child_of` (int): Only display pages that are children of the specified page ID.
- `exclude` (string): Comma-separated list of page IDs to exclude.
- `include` (string): Comma-separated list of page IDs to include.
- `depth` (int): Determines how many levels of hierarchy to show (default is 0, which shows all levels).

#### **Example**:
```php
wp_list_pages( array(
    'title_li'    => '', // Removes the default title
    'sort_column' => 'post_title',
    'exclude'     => '2,4' // Exclude pages with IDs 2 and 4
) );
```

This code will output an unordered list (`<ul>`) of page links sorted by title, excluding pages with IDs 2 and 4.

### **Key Differences**:

1. **Output**: 
   - `get_pages()` returns an array of page objects, which you can loop through and display however you like.
   - `wp_list_pages()` automatically generates an HTML list (`<ul>` and `<li>`) of pages.

2. **Customization**: 
   - `get_pages()` gives you more flexibility to manipulate page data before displaying it. 
   - `wp_list_pages()` is more straightforward if you just need a simple list of page links.

### When to Use Which?
- Use `get_pages()` when you need to manipulate the page data programmatically or want to display pages in a custom format.
- Use `wp_list_pages()` when you want a simple hierarchical list of pages to be displayed without much customization.

Let me know if you need code examples for a specific scenario or further clarification!