## 📌 **`register_rest_field()` in WordPress**

`register_rest_field()` is a function in WordPress used to add custom fields to the REST API responses. This allows developers to include additional data in API responses for specific post types, taxonomies, or user objects, making the API more flexible and tailored for custom use cases.

---

### **Purpose of `register_rest_field()`**

The primary purpose of `register_rest_field()` is to extend the default REST API schema by:

1. **Adding Custom Data**: Including extra data fields in the REST API output for posts, terms, users, or any custom object.
2. **Enhancing Integration**: Making the WordPress REST API more versatile for custom web applications, headless WordPress setups, or mobile apps.
3. **Customization of Schema**: Defining data types, sanitization, and validation for custom fields in API responses.

---

### **Core Concepts**

1. **Target Object Type**:
   - `post`
   - `term`
   - `user`
   - Any custom object type.

2. **Schema Configuration**:
   - **Field Name**: The key under which the data appears in the REST API response.
   - **Schema Definition**: Specifies how the field should behave (e.g., its data type and visibility).

3. **Custom Callbacks**:
   - `get_callback`: Retrieves the value for the field to display in the API response.
   - `update_callback`: Handles how data is stored/updated when the field is modified via the API.
   - `schema`: Defines the structure and data type for the field.

---

### **Basic Syntax**

```php
register_rest_field( 
    string|array $object_type,
    string $attribute,
    array $args = []
);
```

- **`$object_type`**: The object (Post Type) type to extend (e.g., `'post'`, `'user'`, `'term'`).
- **`$attribute`**: The field name (e.g., `'custom_meta'`).
- **`$args`**: Array of arguments:
  - `get_callback` (callable): A function to fetch the field data.
  - `update_callback` (callable): A function to update the field data.
  - `schema` (array): Defines the field structure for documentation and validation.

---

### **Example: adding new property "author name" to the RestAPI JSON data**

```php
function university_custom_rest() {
  register_rest_field('post', 'authorName', array(
    'get_callback' => function() {return get_the_author();}
  ));
}

add_action('rest_api_init', 'university_custom_rest');

```

---

### **Example: Adding a Custom Meta Field to a Post**

Suppose you have a post meta field named `_featured_post` and want it to appear in the REST API.

#### Code:

```php
function add_featured_meta_to_rest() {
    register_rest_field(
        'post', // Object type.
        'featured_post', // Field name.
        [
            'get_callback'    => function ( $post ) {
                return get_post_meta( $post['id'], '_featured_post', true );
            },
            'update_callback' => function ( $value, $post ) {
                return update_post_meta( $post->ID, '_featured_post', $value );
            },
            'schema'          => [
                'description' => 'Featured post meta value',
                'type'        => 'string',
                'context'     => ['view', 'edit'],
            ],
        ]
    );
}
add_action( 'rest_api_init', 'add_featured_meta_to_rest' );
```

#### Output in REST API:

When fetching a post using `/wp-json/wp/v2/posts/<id>`, you’ll see:

```json
{
    "id": 1,
    "title": "Sample Post",
    "featured_post": "true" // Custom field
}
```

---

### **Use Cases in Practice**

1. **Customizing API Responses**:
   - Add calculated data (e.g., average ratings for a product post).
   - Include post meta or term meta in responses.

2. **Headless CMS**:
   - Deliver tailored content for front-end frameworks like React, Angular, or Vue.

3. **Mobile App Development**:
   - Serve custom data to mobile apps via API without modifying core schemas.

4. **Enhancing Plugin Functionality**:
   - Extend APIs for plugins like WooCommerce or Advanced Custom Fields (ACF).

---

### **Advanced Example: Adding Multiple Fields**

Add custom fields to a custom post type `book` for the fields `author_name` and `rating`.

#### Code:

```php
function add_book_fields_to_rest() {
    register_rest_field(
        'book',
        'author_name',
        [
            'get_callback' => function ( $post ) {
                return get_post_meta( $post['id'], 'author_name', true );
            },
            'update_callback' => function ( $value, $post ) {
                return update_post_meta( $post->ID, 'author_name', $value );
            },
            'schema' => [
                'description' => 'Name of the book author',
                'type' => 'string',
                'context' => ['view', 'edit'],
            ],
        ]
    );

    register_rest_field(
        'book',
        'rating',
        [
            'get_callback' => function ( $post ) {
                return get_post_meta( $post['id'], 'rating', true );
            },
            'update_callback' => function ( $value, $post ) {
                return update_post_meta( $post->ID, 'rating', $value );
            },
            'schema' => [
                'description' => 'Rating of the book',
                'type' => 'number',
                'context' => ['view', 'edit'],
            ],
        ]
    );
}
add_action( 'rest_api_init', 'add_book_fields_to_rest' );
```

---

### **Best Practices**

1. **Security**:
   - Validate and sanitize all input using `update_callback` to prevent injection attacks.
   - Restrict sensitive data from being exposed in the API.

2. **Use Schema Properly**:
   - Define clear and descriptive schemas for API consumers to understand your data structure.

3. **Leverage Context**:
   - Use the `context` parameter to control visibility (e.g., `'view'` or `'edit'`).

4. **Optimize Performance**:
   - Use caching for heavy computations in `get_callback`.

---

By using `register_rest_field()`, you can significantly enhance WordPress REST API capabilities, allowing for a more customized and powerful interaction with your content and data.