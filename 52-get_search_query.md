## 📌 `get_search_query()` in WordPress

The `get_search_query()` function in WordPress is a handy tool for retrieving the current search query entered by a user. Below is a detailed explanation of its purpose, core concepts, and practical use cases.

---

### **Purpose**
**The primary purpose of `get_search_query()` is to fetch the search term(s) that users input into the search form. This function can be used anywhere within your theme or plugin to display or utilize the search query.**

---

### **Core Concepts**
1. **Function Signature**: 
   ```php
   string get_search_query( bool $escaped = true )
   ```
   - **$escaped (bool)**: Determines whether the query is escaped for safe output. Default is `true`. When set to `true`, it ensures that the search query is properly escaped to prevent XSS attacks.

2. **Return Value**:
   - The function returns the search query as a string. If there is no search query, it will return an empty string.

3. **Context**:
   - Typically used on search result pages (`is_search()`).

4. **Escaping**:
   - The function escapes the returned value for safe usage in HTML if `$escaped` is `true`. When using the query in raw format (e.g., inside attributes or custom logic), pass `false` to avoid double-escaping.

---

### **Practical Use Cases**
Here are some common scenarios where `get_search_query()` proves useful:

#### **1. Display the Search Term on the Search Results Page**
You can show the search term to users to confirm what they searched for:
```php
<?php if ( is_search() ) : ?>
    <h1>Search Results for: "<?php echo get_search_query(); ?>"</h1>
<?php endif; ?>
```

#### **2. Pre-fill the Search Form Input Field**
You can ensure that the search box retains the search term after submission:
```php
<form method="get" action="<?php echo esc_url( home_url( '/' ) ); ?>">
    <input type="text" name="s" value="<?php echo get_search_query(); ?>" placeholder="Search...">
    <button type="submit">Search</button>
</form>
```

#### **3. Modify or Customize Search Functionality**
You can use `get_search_query()` in combination with other functions or filters to enhance search functionality:
- **Filter Example**:
   Highlight the search term in the search results:
   ```php
   function highlight_search_query( $text ) {
       if ( is_search() ) {
           $query = get_search_query( false ); // Unescaped
           if ( ! empty( $query ) ) {
               $text = preg_replace( '/(' . preg_quote( $query, '/' ) . ')/iu', '<span class="highlight">$1</span>', $text );
           }
       }
       return $text;
   }
   add_filter( 'the_excerpt', 'highlight_search_query' );
   add_filter( 'the_title', 'highlight_search_query' );
   ```

#### **4. Use in Widgets or Custom Plugins**
Retrieve the search term to create a custom widget or plugin feature:
```php
function custom_search_widget() {
    if ( is_search() ) {
        echo '<p>You searched for: ' . esc_html( get_search_query() ) . '</p>';
    }
}
add_action( 'wp_footer', 'custom_search_widget' );
```

#### **5. Logging Search Queries**
Log or track search queries for analytics or optimization:
```php
function log_search_queries() {
    if ( is_search() ) {
        $search_term = get_search_query( false ); // Unescaped
        error_log( 'Search Query: ' . $search_term );
    }
}
add_action( 'wp_head', 'log_search_queries' );
```

---

### **Best Practices**
1. **Escape the Query**: Always use the escaped query for output to ensure safety against cross-site scripting (XSS) attacks.
2. **Avoid Hardcoding**: Use `get_search_query()` instead of hardcoding the search query logic for flexibility and consistency.
3. **Combine with Conditional Tags**: Check if the current page is a search results page using `is_search()` to prevent unnecessary usage elsewhere.

---

### **Related Functions**
1. **`get_query_var('s')`**: Similar to `get_search_query()`, it retrieves the raw search term but doesn't handle escaping.
2. **`is_search()`**: Checks if the current page is a search results page.
3. **`the_search_query()`**: Outputs the search query directly (HTML-escaped).

---

By leveraging `get_search_query()` effectively, you can create a more intuitive and user-friendly search experience while maintaining secure and efficient coding practices.

---