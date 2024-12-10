## 📌 **Understanding `found_posts` in WordPress**

`found_posts` is a key property in WordPress's query system, often used in conjunction with `WP_Query`. **It represents the total number of posts that match a given query, ignoring pagination limits. This property is particularly useful when you need to determine the full number of posts that meet your criteria, such as for custom pagination or statistical displays.**

---

### **Purpose of `found_posts`**

The primary purpose of `found_posts` is to provide the **total count of posts** that match the query arguments. This is especially helpful in scenarios where:

1.❗️ You need to create **custom pagination**.
2.❗️ You want to display the **total number of results** for a query.
3.❗️ You’re working on optimizing or analyzing query performance.

---

### **❗️Core Concepts Behind `found_posts`❗️**

#### 1. **`WP_Query` and Query Results**
   - When you create a new `WP_Query`, it fetches a set number of posts based on the `posts_per_page` argument.
   - For instance, if you set `posts_per_page` to 10, the query retrieves only the first 10 posts.

#### 2. **Total Matching Posts**
   - `found_posts` gives the total number of posts that match the query **before applying the `posts_per_page` limit**.
   - This is useful for determining how many pages of results there are.

#### 3. **Pagination and `max_num_pages`**
   - WordPress calculates the `max_num_pages` property using `found_posts` and the `posts_per_page` value.
   - Formula: `max_num_pages = ceil( found_posts / posts_per_page )`.

#### 4. **Impact of Filters**
   - Filters like `pre_get_posts` or `posts_where` can modify the query and, in turn, the `found_posts` value.
   - Developers can use the `found_posts_query` filter to optimize or alter the behavior of `found_posts`.

---

### **How `found_posts` Works in Practice**

#### Example 1: **Basic Query with `found_posts`**
```php
$args = array(
    'post_type' => 'post',
    'posts_per_page' => 10,
);
$query = new WP_Query( $args );

// Total number of posts matching the query.
$total_posts = $query->found_posts;

echo "Total posts found: " . $total_posts;
```

**Output:** Displays the total count of posts that match the query, even if the current page only shows a subset.

---

#### Example 2: **Custom Pagination Using `found_posts`**
```php
$paged = ( get_query_var('paged') ) ? get_query_var('paged') : 1;

$args = array(
    'post_type' => 'post',
    'posts_per_page' => 5,
    'paged' => $paged,
);

$query = new WP_Query( $args );

// Calculate total pages.
$total_pages = ceil( $query->found_posts / 5 );

echo "Page $paged of $total_pages";
```

**Output:** Allows you to create a custom pagination system with proper page numbers.

---

#### Example 3: **Displaying Results Count**
```php
$args = array(
    'post_type' => 'product',
    'posts_per_page' => 20,
);
$query = new WP_Query( $args );

echo "Showing " . count( $query->posts ) . " of " . $query->found_posts . " products.";
```

**Output:** "Showing 20 of 50 products."

---

### **Use Cases for `found_posts`**

1. **Custom Pagination Systems**
   - When building advanced navigation for archives, category pages, or custom post types.

2. **Search Results Pages**
   - Displaying "X results found for your query."

3. **E-commerce Applications**
   - Showing total products in a category or filtered view.

4. **Custom Dashboards or Admin Panels**
   - For reports that include counts of posts, such as "Total drafts," "Total published posts," etc.

---

### **Optimizing the Use of `found_posts`**

1. **Disable `found_posts` Calculation**
   - If you don't need pagination or the total count, you can optimize performance by setting `'no_found_rows' => true` in the query arguments.
   - Example:
     ```php
     $args = array(
         'post_type' => 'post',
         'posts_per_page' => 10,
         'no_found_rows' => true, // Skips the `found_posts` calculation.
     );
     $query = new WP_Query( $args );
     ```

2. **Filter `found_posts_query`**
   - Developers can use the `found_posts_query` filter to modify the SQL query used for the count calculation.

   ```php
   add_filter( 'found_posts_query', function( $sql, $query ) {
       // Modify the SQL query if needed.
       return $sql;
   }, 10, 2 );
   ```

---

### **Things to Note**

- **Performance:** Calculating `found_posts` involves an additional database query, which can impact performance for large datasets.
- **`no_found_rows` Limitation:** When using `'no_found_rows' => true`, you can't rely on `found_posts` as it won't be populated.

By understanding how `found_posts` works and when to use it, you can harness its power for building dynamic and user-friendly WordPress applications.