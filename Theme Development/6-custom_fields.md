## 📌 Custom Fields in WordPress

Custom fields in WordPress allow you to add extra information, or metadata, to your posts, pages, or custom post types. They are a flexible way to store additional data beyond the default content provided by WordPress. Here are the core concepts you should understand about custom fields.

#### 1. What Are Custom Fields?

**Custom fields are key-value pairs used to store additional information about a post or page**. WordPress stores this metadata in the `wp_postmeta` database table. For example, if you're building a real estate site, you could use custom fields to add details like price, location, and square footage to each property post.

**Key:** This is the label for the custom field (e.g., `property_price`).

**Value:** This is the actual data stored (e.g., $250,000).

#### 2. How Custom Fields Work

**Custom fields work by assigning metadata to specific posts or pages. Each post can have multiple custom fields. You can use them to display additional data on the front end of your site or to handle backend logic.**

For example, if you're building a movie review site, you could create custom fields for the rating, release year, and genre, which are then displayed on the post page.

#### 3. Creating and Managing Custom Fields

WordPress offers a built-in UI for custom fields in the post editor, but it's disabled by default in the Block Editor (Gutenberg). You can enable it by going to the options at the top-right corner (three-dot menu) and checking the "Custom Fields" option.

Alternatively, you can manage custom fields programmatically using functions like `add_post_meta()`, `get_post_meta()`, and `update_post_meta()`.

#### 4. Displaying Custom Fields on the Front End

To display the data from custom fields on your site, you'll need to edit your theme files. You can use the get_post_meta() function to retrieve and echo the custom field value in your template.

**Example:**

```php
<?php
    $price = get_post_meta(get_the_ID(), 'property_price', true);
    echo 'Price: ' . $price;
?>
```

This example retrieves the custom field with the key `property_price` and displays its value.

#### 5. Custom Fields in Themes and Plugins

**Custom fields can be integrated into themes and plugins to extend functionality.** Many WordPress themes, particularly premium ones, offer custom fields as part of their framework, letting you build rich content types without writing code. Popular plugins like Advanced Custom Fields (ACF), CMB2 (Custom Metaboxes 2) make it even easier to create, manage, and display custom fields with a more intuitive interface.

#### 6. Advanced Usage

You can also use custom fields in combination with:

- **Custom Post Types:** Custom fields can be essential for adding metadata to custom post types like products, portfolios, or properties.
- **Querying Posts:** You can query posts by custom fields using `WP_Query` with meta_query parameters, allowing you to create dynamic pages based on custom field data.

#### 7. Custom Field Types

With plugins like ACF, you can add different types of custom fields, such as text fields, numbers, date pickers, and even file uploads, which provide even more control over the data you're adding.

#### 8. Performance Considerations

Since custom fields are stored as post metadata, overloading posts with excessive custom fields or using them inefficiently can cause performance issues. You should ensure that the custom fields you use are necessary and optimized.

In summary, custom fields provide a powerful way to extend WordPress’s capabilities, enabling you to add custom metadata to your posts or pages, create dynamic templates, and build more complex websites without modifying the core WordPress code.

---

#### `is_post_type_archive()`

Determines whether the query is for an existing post type archive page.

**@params:** Post type or array of posts types to check against.

**@Return:** **_bool_** Whether the query is for an existing post type archive page.

#### `is_main_query(): bool`

Determines whether the query is the default/main/core, (default URL based) based query (NOT Custom Query).

```php
function university_adjust_queries ($query) {
  if(!is_admin() AND is_post_type_archive('events') AND $query->is_main_query()){
    $query->set('posts_per_page', '1');
  }
}
add_action('pre_get_posts', 'university_adjust_queries');
```

---
