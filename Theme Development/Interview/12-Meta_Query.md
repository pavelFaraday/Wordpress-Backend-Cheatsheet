# 📌 Meta Queries

#### Understanding Meta Queries in WordPress

In WordPress, a **meta query** is a specialized type of query used within the `WP_Query` class to  **filter posts based on the value of <u>post meta fields (also known as custom fields).</u>** Post meta fields allow you to store additional information about a post, page, or custom post type, and meta queries make it easy to retrieve content based on this data.

#### Why Use Meta Queries?

Meta queries allow developers to create dynamic and customized content filtering, which is useful in various scenarios like building event calendars, displaying product listings, creating advanced search functionalities, and much more. By using meta queries, you can:
- Filter posts based on custom data.
- Retrieve and display relevant information efficiently.
- Provide users with tailored content based on their preferences or specific criteria.

### Core Concepts of Meta Queries

Meta queries work within the `WP_Query` class to filter results based on meta fields. Here's a look at the essential parameters of a meta query:


### Common Parameters:
- **key**: The meta key to filter by.
- **value**: The value you're looking for in that meta key.
- **compare**: Comparison operator (e.g., `=`, `!=`, `LIKE`, `NOT LIKE`, `>`, `<`, etc.).
- **type**: The type of data being compared (e.g., `NUMERIC`, `CHAR`, `DATE`).
  
#### 1. **Key (`key`)**
The `key` parameter specifies the **meta key** (custom field name) that you want to filter by. It's the name of the meta field you're targeting in your query.

#### 2. **Value (`value`)**
The `value` parameter is the actual value you're searching for within the specified meta key. It can be a specific value or a range, depending on your use case.

#### 3. **Comparison Operator (`compare`)**
The `compare` parameter defines the operator used for comparison between the meta key's value and the specified value. Common operators include:
- `=` (equal to)
- `!=` (not equal to)
- `>`, `<` (greater than, less than)
- `>=`, `<=` (greater than or equal to, less than or equal to)
- `LIKE` (partial match)
- `NOT LIKE` (no partial match)
- `IN` (matches one of the values in an array)
- `NOT IN` (does not match any values in an array)

#### 4. **Type (`type`)**
The `type` parameter specifies the data type for the comparison. Common data types include:
- `NUMERIC` (for numbers)
- `CHAR` (for strings)
- `DATE` (for dates)
- `DATETIME` (for timestamps)
- `BINARY` (for binary data)

Setting the `type` parameter ensures accurate comparison based on the data format.

#### Practical Use Cases for Meta Queries

```php
// Hide event if it's date is in the past  (show event if event_date >= Today)
$today = date('Ymd'); // Today
$homepageEvents = new WP_Query(array(
    'posts_per_page' => -1,
    'post_type' => 'events',
    'meta_key' => 'event_date', // 'event_date' - name of the custom field
    'orderby' => 'meta_value_num',  // Date is number, so sort with Date ('event_date') ASC order
    'meta_query' => array(
        array(
        'key' => 'event_date', // meta_key -> name of the custom field
        'compare' => '>=', // Comparision
        'value' => $today,
        'type' => 'numeric' // We should compare only numeric values (date)
        )
    ), // All this Array logic means: Only show event posts if event_date >= Today Date
    'order' => 'ASC',
));
```

Meta queries have several real-world applications in WordPress development. Let's explore some common scenarios:

#### 1. **Event Management**
To display upcoming events, you might use a meta query to filter events based on the event date, ensuring only future events are shown:

```php
$today = date('Ymd'); // Today's date
$events = new WP_Query(array(
    'post_type' => 'events',
    'meta_key' => 'event_date',
    'orderby' => 'meta_value_num',
    'meta_query' => array(
        array(
            'key' => 'event_date',
            'value' => $today,
            'compare' => '>=',
            'type' => 'NUMERIC'
        )
    ),
    'order' => 'ASC',
));
```

This query only shows events where the event date is today or later.

#### 2. **E-commerce Product Filtering**
Meta queries are ideal for building product catalogs in e-commerce sites. For example, to display products priced between 10 and 50$:

```php
$products = new WP_Query(array(
    'post_type' => 'product',
    'meta_query' => array(
        array(
            'key' => 'price',
            'value' => array(10, 50),
            'compare' => 'BETWEEN',
            'type' => 'NUMERIC'
        )
    )
));
```

This retrieves all products within the specified price range.

#### 3. **Custom User Search**
If you have a directory or listing of user profiles, meta queries can help filter users based on custom fields like skills or location:

```php
$developers = new WP_Query(array(
    'post_type' => 'user',
    'meta_query' => array(
        array(
            'key' => 'skills',
            'value' => 'WordPress',
            'compare' => 'LIKE'
        )
    )
));
```

This query returns all users with 'WordPress' listed as one of their skills.

#### 4. **Real Estate Listings**
For real estate websites, meta queries are useful to filter properties based on criteria like price, number of bedrooms, or availability:

```php
$properties = new WP_Query(array(
    'post_type' => 'property',
    'meta_query' => array(
        array(
            'key' => 'bedrooms',
            'value' => 3,
            'compare' => '>=',
            'type' => 'NUMERIC'
        ),
        array(
            'key' => 'availability',
            'value' => 'available',
            'compare' => '='
        )
    )
));
```

This displays all properties with at least three bedrooms that are currently available.

### Combining Multiple Meta Queries

Meta queries also support combining multiple conditions using the `relation` parameter (`AND`, `OR`) to filter posts based on more complex logic:

```php
$args = array(
    'post_type' => 'events',
    'meta_query' => array(
        'relation' => 'AND',
        array(
            'key' => 'event_date',
            'value' => $today,
            'compare' => '>=',
            'type' => 'DATE'
        ),
        array(
            'key' => 'event_location',
            'value' => 'New York',
            'compare' => '='
        )
    )
);
$events = new WP_Query($args);
```

This query retrieves events happening today or later in New York.

### Best Practices for Using Meta Queries

1. **Use Proper Indexing:** Using indexed meta keys for frequently queried fields can improve database performance.
2. **Avoid Unnecessary Meta Queries:** If possible, consider using taxonomies or other WordPress features instead of meta queries, as they are more efficient for filtering.
3. **Optimize Query Performance:** Use data types (`NUMERIC`, `DATE`, etc.) in your meta queries to ensure accurate and faster comparisons.

### Final Thoughts

Meta queries are a robust feature in WordPress that enables developers to retrieve and display content dynamically based on post meta fields (custom field data). Whether you’re building an event calendar, an e-commerce shop, or a user directory, mastering meta queries can help you deliver highly customized solutions to your WordPress projects.
