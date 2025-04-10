##  📌 `paginate_links()` in WordPress

In WordPress, `pagination_list()` and `paginate_links()` are both related to creating pagination, which allows users to navigate through different pages of posts, comments, or other sections of a website. These functions are critical in creating a seamless and user-friendly experience, particularly on content-heavy websites.

Let’s break down each of these functions:

## 1. `paginate_links()`

### **Purpose**
The `paginate_links()` function is used to generate pagination links for archive pages, post pages, or any custom queries. It's a built-in WordPress function that returns an HTML string of paginated links. This function offers a flexible way to create navigation for multi-page content.

❗️This is general WP pagination & only works with default queries, that WP makes on its own tied to the current URL.❗️

### **Core Concepts**
- **Pagination in WordPress**: Pagination refers to dividing content into pages. It’s essential for improving user experience and website performance, as loading large amounts of content on a single page can be inefficient.
- **Context**: Typically used in archive pages, like blog listings, search results, or custom query results where posts need to be split into several pages.

### **Syntax**
```php
paginate_links( array $args = array() );
```
**Parameters**
- **$args** *(array)*: It accepts an array of arguments that customize the output. The common parameters include:
  - `base` *(string)*: The base URL for pagination links.
  - `format` *(string)*: The URL format for the page number.
  - `current` *(int)*: The current page number.
  - `total` *(int)*: The total number of pages.
  - `show_all` *(bool)*: Whether to show all pages (default: false).
  - `end_size` *(int)*: Number of pages at the beginning and end of pagination links (default: 1).
  - `mid_size` *(int)*: Number of pages on either side of the current page (default: 2).
  - `prev_text` *(string)*: Text for the "Previous" link (default: '&laquo; Previous').
  - `next_text` *(string)*: Text for the "Next" link (default: 'Next &raquo;').

### **Example Use Case**
Let’s say you want to add pagination links on your blog’s archive page:
```php
global $wp_query;

$big = 999999999; // an unlikely integer

$pagination_args = array(
    'base'      => str_replace($big, '%#%', esc_url(get_pagenum_link($big))),
    'format'    => '?paged=%#%',
    'current'   => max(1, get_query_var('paged')),
    'total'     => $wp_query->max_num_pages,
    'prev_text' => __('&laquo; Previous'),
    'next_text' => __('Next &raquo;'),
);

echo paginate_links($pagination_args);
```

### **Purpose & Benefits**
- **Customizable and Flexible**: You can tailor the pagination’s appearance and behavior to match your theme’s needs.
- **SEO-Friendly Links**: Generates clean and SEO-friendly URLs for page numbers.
- **Performance and UX**: Helps in dividing content-heavy sections into multiple pages, improving load times and providing a more digestible reading experience.
