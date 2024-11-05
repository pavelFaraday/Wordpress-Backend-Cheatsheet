## 📌 Other functions in WordPress

These functions are essential for building dynamic, user-friendly WordPress sites, enabling developers to access and display various types of content effectively.

1. **`the_author_posts_link()`**: 
   - Outputs a link to the author archive page.
   - Displays the author’s name as a clickable link, allowing visitors to view all posts by that author.
   - Typically used in single post templates.

2. **`get_the_category_list()`**: 
   - Retrieves a formatted list of categories associated with the current post.
   - Allows customization of the separator (e.g., commas, slashes).
   - Useful for displaying categories in a post footer or header.

   **Example**: 
   ```php
   <p><?php echo get_the_category_list(', '); ?></p>
   ```

3. **`get_the_ID()`**: 
   - Returns the unique ID of the current post or page within the Loop.
   - Useful for querying related content or for database operations.

   **Example**: 
   ```php
   <?php echo get_the_ID(); ?> // Outputs post ID, e.g., 16
   ```

4. **`wp_get_post_parent_id()`**: 
   - Fetches the parent post’s ID if the current post is a child; otherwise, returns 0.
   - Helpful for hierarchical post types like pages, allowing you to link back to parent content.

5. **`get_permalink()`**: 
   - Retrieves the complete URL for the current post or for a specific post ID.
   - Useful for creating links to posts dynamically.

6. **`the_archive_title()`**: 
   - Displays the title of the current archive page, adapting to the queried object (category, tag, etc.).
   - Automatically formats the title based on the context, enhancing user experience.

7. **`the_archive_description()`**: 
   - Outputs a description of the current archive, providing additional context (e.g., for categories or tags).
   - Can be set in the WordPress admin area and displayed alongside archive titles.

8. **`get_post_type_archive_link()`**: 
   - Returns the permalink to the archive page for a specific post type.
   - Useful for custom post types to create links to their archives.

   **Example**: 
   ```php
   get_post_type_archive_link('events');
   ```

9. **`single_cat_title()`**: 
   - Displays or retrieves the title of the current category archive.
   - Can be used in templates to provide context to users about the content they are viewing.

10. **`get_post_type()`**: 
   - Returns the post type (e.g., 'post', 'page', or custom types) of the current post or a specified post.
   - Useful for conditional checks in theme or plugin development.
  
11.  **`the_post_thumbnail();`**
   - Displays the post thumbnail.

12. **`the_post_thumbnail_url()`**
   - Displays the post thumbnail URL.

13. How to use `after_setup_theme` action hook
```php
function university_features() {
  // Register Different Menu Locations
  register_nav_menu('headerMenuLocation', 'Header Menu Location');
  register_nav_menu('footerLocationOne', 'Footer Location One');
  register_nav_menu('footerLocationTwo', 'Footer Location Two');

  // Register theme support for a various features.
  add_theme_support('title-tag');
  add_theme_support('post-thumbnails');

  // Register a new/additional image size.
  add_image_size('professorLandscape', 400, 260, true);
  add_image_size('professorPortraite', 480, 650, true);
}
add_action('after_setup_theme', 'university_features');

```
