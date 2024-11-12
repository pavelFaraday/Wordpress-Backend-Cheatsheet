## 📌 `get_query_var()`

Retrieves the value of a query variable in the WP_Query class.

The `get_query_var()` function in WordPress is a versatile utility used to retrieve query variables. These query variables are set within the WP_Query class, which powers the WordPress query system. Essentially, this function allows developers to access specific variables from the main query or custom queries.

---

We use it for Pagination in Custom Queries.

```php
$pastEvents = new WP_Query(array(
    'paged' => get_query_var('paged', 1),
    // another code..
));
```

⬆️⬆️⬆️ Works with ⬇️⬇️⬇️

```php
<?php echo paginate_links(array (
    'total' => $pastEvents->max_num_pages
)); ?>
```

---

## Purpose of `get_query_var()`
**The primary purpose of `get_query_var()` is to retrieve the value of a public query variable (query var) in WordPress. When a query is run, WordPress stores variables that represent the parts of a URL or request. This function taps into that storage and retrieves the variable's value for further use in themes or plugins.**

**For example, if your URL contains a query like `?category_name=travel`, you can use `get_query_var()` to retrieve the value of `category_name`.**

---

## Core Concepts of `get_query_var()`

1. **Query Variables**: Query variables (often called "query vars") are parameters passed in the URL or created programmatically. They are typically defined in the URL or registered through WordPress hooks like `pre_get_posts`. For example, when you visit `https://example.com/?author=1`, the variable `author` becomes available within the WordPress query.

2. **WP_Query Integration**: Query vars are tied closely to WordPress’ `WP_Query` class. This is what powers the queries that retrieve content from the database, including posts, pages, authors, categories, and custom post types. `get_query_var()` enables developers to access the values set by `WP_Query`.

3. **Public Query Vars**: Only public query variables can be accessed using `get_query_var()`. Public query variables include pre-defined variables like `author`, `paged`, `category_name`, etc. You can also create custom public query vars using `add_query_var`.

4. **Customization**: Developers can add and customize query vars using filters like `query_vars`, allowing retrieval of custom data with `get_query_var()`.

---

## Syntax of `get_query_var()`

```php
$value = get_query_var( $var, $default_value );
```

- **$query_var** (string, required): The name of the query variable to retrieve.
- **$default_value** (mixed, optional): The default value to return if the query variable is not set.

- **@return:** Contents of the query variable.

### Example:

```php
$paged = get_query_var('paged', 1);
```

This code retrieves the current page number if it’s set. Otherwise, it defaults to 1.

---

## Practical Use Cases of `get_query_var()`

### 1. **Pagination in Custom Loops**
In WordPress, when dealing with custom queries, pagination is a common use case. For example, you may want to retrieve the current page in a custom loop:

```php
$paged = (get_query_var('paged')) ? get_query_var('paged') : 1;
$args = array(
    'post_type' => 'portfolio',
    'posts_per_page' => 10,
    'paged' => $paged,
);

$custom_query = new WP_Query($args);
if ($custom_query->have_posts()) {
    while ($custom_query->have_posts()) {
        $custom_query->the_post();
        // Your custom post output
    }
    // Pagination logic here
}
```

### 2. **Filtering Custom Post Types by a Custom Query Variable**
Suppose you have a custom post type called `events`, and you want to filter it by a custom query variable `event_type`. Here’s how it works:

#### 1. Register the Query Var:
```php
function custom_register_query_var($vars) {
    $vars[] = 'event_type';
    return $vars;
}
add_filter('query_vars', 'custom_register_query_var');
```

#### 2. Use it in a Query:
```php
$event_type = get_query_var('event_type', 'default_type'); // 'default_type' is the fallback
$args = array(
    'post_type' => 'events',
    'meta_key' => 'event_type',
    'meta_value' => $event_type
);
$events_query = new WP_Query($args);
if ($events_query->have_posts()) {
    while ($events_query->have_posts()) {
        $events_query->the_post();
        // Output event details
    }
}
```

### 3. **Handling Rewrite Rules for Custom Endpoints**
`get_query_var()` is heavily used in custom rewrite rules to capture data from custom URLs. For instance, if you’re creating a custom URL structure like `/portfolio/project-name/`, you can set a query variable `project-name` and retrieve it later in your template:

#### 1. Add a Rewrite Rule and Query Var:
```php
function add_custom_rewrite_rule() {
    add_rewrite_rule('^portfolio/([^/]*)/?', 'index.php?pagename=portfolio&project_name=$matches[1]', 'top');
}
add_action('init', 'add_custom_rewrite_rule');

function add_custom_query_vars($vars) {
    $vars[] = 'project_name';
    return $vars;
}
add_filter('query_vars', 'add_custom_query_vars');
```

#### 2. Use `get_query_var()` to Access the Query Var:
```php
$project_name = get_query_var('project_name');
if ($project_name) {
    // Output project information based on $project_name
}
```

### 4. **Checking if a Specific Query Var Exists**
Sometimes, you might just need to check if a specific query variable is set. For example, redirecting users based on a query var or making conditional changes:

```php
if (get_query_var('category_name') == 'news') {
    // Special styling or behavior for the 'news' category
}
```

### 5. **Creating Custom Admin Pages or Dashboard Widgets**
If you have a custom admin page where specific query variables dictate what content is displayed, `get_query_var()` can help manage these changes. For example, switching between tabs or data views based on the query variable in the admin area.

---

## Important Points to Remember
- **Public vs Private Query Vars**: Only public query variables can be retrieved using `get_query_var()`. If you define private or custom variables in your WP_Query, they won't be accessible unless explicitly registered as public.
- **Adding Custom Query Vars**: Use the `query_vars` filter to add your custom variables so they are accessible in WordPress queries.
- **Interaction with WP_Rewrite**: `get_query_var()` works well with custom rewrite rules, enabling you to create user-friendly URLs.

---

## Summary
The `get_query_var()` function is an essential tool for developers working with WordPress. It allows for seamless retrieval of query variables, both default and custom, making it ideal for creating dynamic content, custom post type filtering, custom pagination, and much more.

In practice, this function shines in scenarios involving URL manipulation, custom rewrites, and conditional logic based on query values. By understanding its purpose and core concepts, developers can leverage it to build more flexible and powerful WordPress websites.