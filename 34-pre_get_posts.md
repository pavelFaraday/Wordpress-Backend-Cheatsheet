## 📌 `pre_get_posts` hook in WordPress

The `pre_get_posts` function in WordPress is an essential hook that allows developers to **modify or extend the default behavior of WordPress queries. It’s a powerful way to change how posts are retrieved, offering a wide variety of use cases, from customizing archive pages to creating conditional logic for specific types of content.**

### Purpose of `pre_get_posts`

The purpose of `pre_get_posts` is to give developers a way to alter the query that WordPress uses to fetch posts and other content. **This hook is executed after WordPress sets up the default query, but before the query is executed in the database.** It enables fine-grained control over the query parameters, such as modifying post types, including or excluding categories, changing the number of posts displayed, and more.

### Core Concepts of `pre_get_posts`

1. **Hook Type:** `pre_get_posts` is an **action hook** in WordPress.
2. **Triggered:** It fires after WordPress has parsed the URL and set up the main query, but before executing it.
3. **Modifies WP_Query:** It allows developers to modify the properties of a `WP_Query` object, which means that the modifications will affect what posts are returned.
4. **Available Query Properties:** You can change attributes like `post_type`, `category_name`, `meta_query`, `tax_query`, `orderby`, `posts_per_page`, and many more.

### **Syntax of `pre_get_posts`**

To use this hook, you need to write a function and attach it to the `pre_get_posts` action in your theme’s `functions.php` file or a custom plugin.

```php
function custom_modify_main_query( $query ) {
    // Ensure that we are modifying the main query and not other custom queries
    if ( $query->is_main_query() && ! is_admin() ) {
        // Add your customizations here
    }
}
add_action( 'pre_get_posts', 'custom_modify_main_query' );
```

### Detailed Explanation of Key Concepts

- **Main Query vs. Custom Queries:** The primary query that determines what content to display on a page is called the "main query." WordPress uses it to determine what posts or pages to load based on the URL requested. Developers should target only the main query by using `$query->is_main_query()` to avoid unintentionally affecting custom queries.

- **Conditionally Altering the Query:** The conditional checks like `is_home()`, `is_archive()`, `is_search()`, `is_category()`, etc., help in targeting specific pages or requests. These functions return `true` or `false` based on the type of page being viewed, allowing you to alter the query contextually.

- **Avoid Altering the Admin Panel:** Use `!is_admin()` to ensure that you are only modifying queries on the frontend and not interfering with admin functionalities.

### Practical Use Cases

#### 1. **Change the Number of Posts on the Homepage**

By default, WordPress shows a specific number of posts (set in Settings > Reading). But using `pre_get_posts`, you can dynamically change this based on certain conditions:

```php
function custom_posts_per_page( $query ) {
    if ( $query->is_main_query() && ! is_admin() && $query->is_home() ) {
        $query->set( 'posts_per_page', 5 );
    }
}
add_action( 'pre_get_posts', 'custom_posts_per_page' );
```

#### 2. **Include Custom Post Types in Archives**

Suppose you want to include custom post types like `portfolio` in the main blog archive:

```php
function include_custom_post_types_in_archive( $query ) {
    if ( $query->is_main_query() && ! is_admin() && ( $query->is_archive() || $query->is_home() ) ) {
        $query->set( 'post_type', array( 'post', 'portfolio' ) );
    }
}
add_action( 'pre_get_posts', 'include_custom_post_types_in_archive' );
```

#### 3. **Modify the Search Query**

You can control what post types appear in the search results using `pre_get_posts`. For example, you can limit the search results to just posts and pages:

```php
function modify_search_filter( $query ) {
    if ( $query->is_main_query() && ! is_admin() && $query->is_search() ) {
        $query->set( 'post_type', array( 'post', 'page' ) );
    }
}
add_action( 'pre_get_posts', 'modify_search_filter' );
```

#### 4. **Exclude a Category from the Homepage**

If you want to hide specific categories from the homepage, you can do so like this:

```php
function exclude_category_from_home( $query ) {
    if ( $query->is_main_query() && ! is_admin() && $query->is_home() ) {
        $query->set( 'cat', '-5' ); // Use the ID of the category you want to exclude
    }
}
add_action( 'pre_get_posts', 'exclude_category_from_home' );
```

#### 5. **Custom Order for Taxonomy Terms**

If you have custom taxonomies and you want to sort them in a specific way on an archive page, you can use this hook:

```php
function custom_order_by_taxonomy( $query ) {
    if ( $query->is_main_query() && ! is_admin() && $query->is_tax('your_taxonomy') ) {
        $query->set( 'orderby', 'title' );
        $query->set( 'order', 'ASC' );
    }
}
add_action( 'pre_get_posts', 'custom_order_by_taxonomy' );
```

### Best Practices

1. **Check if it’s the Main Query:** Always ensure you are modifying the main query (`$query->is_main_query()`) and not custom queries made by plugins or additional calls.
2. **Avoid Modifying Admin Queries:** Always use `!is_admin()` to make sure your code doesn’t interfere with backend functionality.
3. **Use Proper Conditional Tags:** Use appropriate conditional checks like `is_home()`, `is_category()`, `is_tag()`, `is_author()`, etc., to target specific areas of the site.

### Summary

The `pre_get_posts` function in WordPress is a vital tool for customizing and fine-tuning how content is displayed on your site. Whether you need to include custom post types, change the number of posts, filter search results, or exclude categories, this hook provides the flexibility to do so. By following best practices and using the appropriate conditional logic, you can effectively leverage `pre_get_posts` to build a more dynamic and responsive WordPress site.