# Wordpress Actions Library



## 1. 📌  `wp_enqueue_scripts` 

`wp_enqueue_scripts` is a key WordPress function that allows developers to properly load scripts (JavaScript files) and styles (CSS files) into a theme or plugin. It helps ensure that assets are added in the correct way, avoiding conflicts, improving performance, and maintaining compatibility with WordPress standards.

### Purpose of `wp_enqueue_scripts`

The primary purpose of `wp_enqueue_scripts` is to manage how and when scripts and styles are loaded on your WordPress site. Using this function ensures that:

- Scripts and styles are added **efficiently**, only where needed.
- Dependencies between scripts are handled properly.
- Conflicts with other themes or plugins are avoided.
- Your assets are loaded in the correct order (e.g., jQuery before your custom script).
- Duplicate loading of assets is prevented, boosting performance.
  
By enqueuing scripts and styles, you allow WordPress to take care of some core tasks like versioning, combining, and minifying them as needed.

### Core Concepts

1. **Hooking into `wp_enqueue_scripts`:**  
   The function `wp_enqueue_scripts` should be used with the appropriate WordPress action hook to ensure your scripts are enqueued at the right time.
   ```php
   add_action( 'wp_enqueue_scripts', 'my_custom_script_enqueue' );
   function my_custom_script_enqueue() {
       // Enqueue styles and scripts here
   }
   ```

2. **Script Enqueueing (`wp_enqueue_script`):**  
   Enqueue JavaScript files using the `wp_enqueue_script()` function. It allows you to specify a script's location, version, dependencies, and more.
   ```php
   wp_enqueue_script( 'custom-js', get_template_directory_uri() . '/js/custom.js', array( 'jquery' ), '1.0.0', true );
   ```
   - **'custom-js'** is the handle to reference the script.
   - **`get_template_directory_uri()`** provides the full URL to the `js/custom.js` file.
   - **`array('jquery')`** defines dependencies (loads jQuery first).
   - **`1.0.0`** is the version of the script.
   - **`true`** loads the script in the footer (for better performance).

3. **Style Enqueueing (`wp_enqueue_style`):**  
   Similarly, you can enqueue CSS files with `wp_enqueue_style()`:
   ```php
   wp_enqueue_style( 'custom-style', get_template_directory_uri() . '/css/custom.css', array(), '1.0.0', 'all' );
   ```
   - **'custom-style'** is the handle for the style.
   - **`get_template_directory_uri()`** gives the URL to `css/custom.css`.
   - **`array()`** defines dependencies (empty array if no dependencies).
   - **`1.0.0`** is the version number.
   - **`'all'`** specifies that the style applies to all media types.

4. **Dependency Management:**  
   Enqueuing scripts or styles with dependencies ensures that WordPress knows which files must be loaded first. For example, if your script relies on jQuery, defining it in the dependencies array ensures that jQuery is loaded first:
   ```php
   wp_enqueue_script( 'my-script', get_template_directory_uri() . '/js/my-script.js', array( 'jquery' ), '1.0.0', true );
   ```

5. **Version Control:**  
   Adding a version number helps with cache-busting. If the file changes, browsers can detect it via the new version number and download the latest version, ensuring users don’t see an outdated file.

6. **Load in Footer:**  
   By passing `true` as the last argument in `wp_enqueue_script()`, your script is loaded in the footer, reducing page load time. Critical scripts, such as jQuery, can be loaded in the header, but non-essential or large scripts should load in the footer to improve performance.

7. **Localization (`wp_localize_script`):**  
   You can pass dynamic data (like URLs, nonces, or user settings) from PHP to JavaScript using the `wp_localize_script()` function.
   ```php
   wp_localize_script( 'my-script', 'myScriptData', array( 'ajax_url' => admin_url( 'admin-ajax.php' ) ) );
   ```

### Use Cases in Practice

1. **Adding Custom Scripts and Styles in Themes:**
   In a WordPress theme, `wp_enqueue_scripts` is used in the `functions.php` file to load stylesheets and scripts, ensuring proper functionality across the site.
   ```php
   function theme_enqueue_scripts() {
       wp_enqueue_style( 'theme-main-style', get_stylesheet_uri() );
       wp_enqueue_script( 'theme-custom-js', get_template_directory_uri() . '/js/custom.js', array(), '1.0.0', true );
   }
   add_action( 'wp_enqueue_scripts', 'theme_enqueue_scripts' );
   ```

2. **Loading Scripts Conditionally:**
   You can use conditions to load scripts only on certain pages or post types, improving performance by not loading unnecessary assets.
   ```php
   function conditionally_enqueue_scripts() {
       if ( is_single() && 'post' === get_post_type() ) {
           wp_enqueue_script( 'single-post-js', get_template_directory_uri() . '/js/single-post.js', array(), '1.0.0', true );
       }
   }
   add_action( 'wp_enqueue_scripts', 'conditionally_enqueue_scripts' );
   ```

3. **Loading Assets for Plugins:**
   When developing a plugin, using `wp_enqueue_scripts` ensures that the plugin’s assets don’t conflict with the theme or other plugins. You typically use a similar pattern in the main plugin file:
   ```php
   function my_plugin_enqueue_scripts() {
       wp_enqueue_style( 'my-plugin-style', plugin_dir_url( __FILE__ ) . 'css/plugin-style.css' );
       wp_enqueue_script( 'my-plugin-script', plugin_dir_url( __FILE__ ) . 'js/plugin-script.js', array(), '1.0.0', true );
   }
   add_action( 'wp_enqueue_scripts', 'my_plugin_enqueue_scripts' );
   ```

4. **Handling Dependencies and External Scripts:**
   You can also enqueue external libraries or scripts, like Google Fonts or a CDN-hosted version of jQuery, in the same way:
   ```php
   wp_enqueue_style( 'google-fonts', 'https://fonts.googleapis.com/css?family=Roboto:400,700' );
   wp_enqueue_script( 'jquery-cdn', 'https://code.jquery.com/jquery-3.6.0.min.js', array(), '3.6.0', true );
   ```

5. **Loading Scripts for the Admin Area:**
   To enqueue scripts and styles in the WordPress admin, you should use the `admin_enqueue_scripts` action hook:
   ```php
   function admin_enqueue_custom_scripts() {
       wp_enqueue_script( 'admin-custom-js', plugin_dir_url( __FILE__ ) . 'js/admin.js', array( 'jquery' ), '1.0.0', true );
   }
   add_action( 'admin_enqueue_scripts', 'admin_enqueue_custom_scripts' );
   ```

### Best Practices

- **Always Enqueue, Never Hardcode:** Avoid hardcoding `<script>` or `<link>` tags directly in template files; use `wp_enqueue_scripts` to load them properly.
  
- **Use Proper Handles:** When enqueuing scripts and styles, always use unique handles. This prevents accidental conflicts with other themes or plugins.
  
- **Only Load What’s Necessary:** Enqueue only the assets required for a specific page, post, or functionality. This enhances the performance of the site by reducing unnecessary HTTP requests.
  
- **Utilize Versioning:** Always specify a version number for scripts and styles to ensure proper cache management, especially when updating assets.


In summary, `wp_enqueue_scripts` is an essential function for WordPress developers to manage scripts and styles efficiently, avoid conflicts, and ensure optimal performance. By following best practices and understanding the core concepts, you can create better, faster, and more maintainable WordPress sites.


---

## 2. 📌  `after_setup_theme`

The `after_setup_theme` action hook in WordPress is an essential tool for theme developers. **It is triggered after the theme is initialized but before most other setup tasks, like rendering content.** This makes it an ideal place to define essential theme features, load text domains for translation, register navigation menus, add theme support for features like post thumbnails, and more. Let’s dive into the details, core concepts, purpose, and practical use cases.

### 1. **Purpose of `after_setup_theme`**

The `after_setup_theme` hook is primarily used to set up the basic structure and features of a WordPress theme. It provides developers with a safe point to add actions and filters or declare theme support before the WordPress front end or back end is fully initialized. **Since this hook runs early in the theme’s loading process, it is the go-to place for any kind of functionality that should be initialized before content is rendered.**

**Key Purpose:**
- Initialize theme support for core WordPress features.
- Register custom menus, image sizes, and other theme functionalities.
- Load text domains for theme localization and translation.
- Setup features that require early theme initialization.

### 2. **Core Concepts**

#### a. **Theme Initialization**:
**The `after_setup_theme` action is fired during the initialization phase of the theme. This means it runs just after the theme has been loaded but before any content (such as posts or pages) is rendered. This allows you to hook into WordPress early to define features that affect how the theme functions or interacts with WordPress.**

#### b. **Theme Support**:
**WordPress comes with various built-in features, like custom logos, post thumbnails, or custom headers. The `after_setup_theme` hook is where you can add support for these features using the `add_theme_support()` function. This is a key concept, as it allows WordPress to know which features your theme supports and enables them accordingly.**

#### c. **Localization**:
Localization allows you to make your theme translation-ready. By using the `load_theme_textdomain()` function inside the `after_setup_theme` hook, you ensure your theme is ready for translation into multiple languages.

#### d. **Timing**:
**The timing of this hook is crucial. Because it fires after the theme is set up but before WordPress continues with other tasks, it ensures that theme-related actions (such as registering menus, custom features, etc.) are ready before WordPress starts rendering content.**

### 3. **How It Works (Code Breakdown)**

Here is an example of how you can use the `after_setup_theme` hook in your `functions.php` file:

```php
function mytheme_setup() {
    // Add theme support for various features
    add_theme_support('post-thumbnails');
    add_theme_support('custom-logo');
    add_theme_support('title-tag');

    // Register a navigation menu
    register_nav_menus(array(
        'primary' => __('Primary Menu', 'mytheme'),
    ));

    // Load the theme's textdomain for localization
    load_theme_textdomain('mytheme', get_template_directory() . '/languages');
}
add_action('after_setup_theme', 'mytheme_setup');
```

### 4. **Practical Use Cases**

#### a. **Adding Theme Support**:
**The most common use of `after_setup_theme` is to add support for WordPress features that need to be defined at theme setup.** 

For example:

```php
function custom_theme_setup() {
    add_theme_support('custom-logo');  // Custom logos
    add_theme_support('post-thumbnails');  // Featured images
    add_theme_support('automatic-feed-links');  // Automatic RSS feed links
    add_theme_support('html5', array('comment-list', 'comment-form', 'search-form', 'gallery', 'caption'));  // HTML5 support
}
add_action('after_setup_theme', 'custom_theme_setup');
```

#### b. **Registering Menus**:
If your theme includes navigation menus, you can register them in `after_setup_theme`. This ensures that menus are set up early in the theme loading process:

```php
function register_my_menus() {
    register_nav_menus(array(
        'header-menu' => __('Header Menu'),
        'footer-menu' => __('Footer Menu')
    ));
}
add_action('after_setup_theme', 'register_my_menus');
```

#### c. **Localization (Translation)**:
To make your theme translatable into different languages, you load your text domain in `after_setup_theme`. This is crucial for internationalization (i18n) and localization (l10n):

```php
function theme_localization_setup() {
    load_theme_textdomain('mytheme', get_template_directory() . '/languages');
}
add_action('after_setup_theme', 'theme_localization_setup');
```

#### d. **Custom Image Sizes**:
The `after_setup_theme` hook is also where you define custom image sizes for your theme:

```php
function custom_image_sizes() {
    add_image_size('homepage-thumb', 220, 180, true);  // Crop mode
}
add_action('after_setup_theme', 'custom_image_sizes');
```

### 5. **Best Practices**
- **Don’t Delay Theme Setup**: Use `after_setup_theme` for tasks that need to be ready early in the theme lifecycle. Avoid adding functionality that can be handled later (like content rendering hooks).
- **Keep Functions Modular**: Rather than placing all your setup code in a single function, try to break it into modular functions for clarity and reusability.
- **Localization Early**: Ensure the text domain is loaded early using `load_theme_textdomain()` in this hook so that translated strings are available throughout the site.
- **Child Themes**: If you’re creating a child theme, ensure that the `after_setup_theme` hook in the child theme has a lower priority than the parent theme to avoid conflicts.

```php
add_action('after_setup_theme', 'child_theme_setup', 11);  // Priority of 11 ensures it runs after the parent theme
```

### 6. **Common Mistakes**
- **Misusing the Hook**: Don’t use `after_setup_theme` for actions that can be handled in other hooks, such as initializing widgets or enqueuing scripts. Those tasks should be placed in hooks like `widgets_init` or `wp_enqueue_scripts`.
- **Ignoring Localization**: Failing to load text domains in this hook can result in untranslated strings.

### 7. **Summary**

The `after_setup_theme` hook is critical for WordPress theme development, serving as the entry point for theme configuration and feature support. It allows theme developers to:
- Add support for core WordPress features.
- Register navigation menus and custom image sizes.
- Set up localization for translations.
- Define theme behavior early in the WordPress loading process.

When used properly, this hook ensures that the theme is fully functional and ready to interact with WordPress, providing a smooth and optimized user experience.

---

## 3. 📌  `pre_get_posts`

The `pre_get_posts` action in WordPress is a powerful tool that developers use to modify the main query before it is executed. It's a core action hook that allows you to change various aspects of the query that WordPress runs to retrieve posts, pages, or other content types. Here's a detailed look at its purpose, core concepts, and practical use cases.

### 1. **Purpose of `pre_get_posts`**
The `pre_get_posts` action hook is used to modify the parameters of the WordPress query object (`$query`) before the query is executed. By hooking into this action, developers can influence what content is retrieved from the database, including altering post types, taxonomies, meta queries, and more. This helps to customize and optimize the content displayed on a website without directly modifying the theme's template files or the WordPress core.

### 2. **Core Concepts**

- **Main Query vs. Secondary Query**: 
  - The **main query** is the query that WordPress runs automatically to determine what content to display on the current page. This could be a list of posts on the homepage, a single post view, an archive, or a search results page.
  - **Secondary queries** are custom queries created with `WP_Query` or functions like `get_posts()` that are used in specific template files or plugin functionalities. `pre_get_posts` primarily affects the main query.

- **Hook Priority**: 
  - The hook fires after the query object has been created but before it has been executed. This makes it the perfect place to modify the query before fetching the results from the database.

- **Conditional Tag Usage**: 
  - When using `pre_get_posts`, it’s important to target specific types of pages using conditional tags like `is_home()`, `is_archive()`, `is_search()`, `is_admin()`, and so on. This ensures that the modifications only apply to the desired sections of the site.

### 3. **How to Use `pre_get_posts` in Practice**

Below is a sample code snippet illustrating how to use `pre_get_posts`:

```php
function my_custom_pre_get_posts( $query ) {
    // Check if it's the main query and not in the admin dashboard
    if ( $query->is_main_query() && !is_admin() ) {
        // Customize the query for the main blog page
        if ( is_home() ) {
            $query->set( 'posts_per_page', 5 ); // Show 5 posts per page
            $query->set( 'category_name', 'news' ); // Show only posts from the 'news' category
        }

        // Customize the query for search results
        if ( is_search() ) {
            $query->set( 'post_type', 'product' ); // Only show products in search results
            $query->set( 'posts_per_page', 10 ); // Display 10 products per page in search results
        }
    }
}
add_action( 'pre_get_posts', 'my_custom_pre_get_posts' );
```

### 4. **Use Cases for `pre_get_posts`**

- **Altering the Number of Posts Displayed on the Homepage**
  - You can use `pre_get_posts` to change how many posts are shown on the homepage without altering your theme’s settings.
  
- **Displaying Custom Post Types on Search Pages**
  - By default, WordPress only searches posts and pages. You can modify the query to include custom post types like products, events, or any other post type you've registered.
  
- **Filtering Posts by Custom Fields or Taxonomies**
  - You can filter posts by custom taxonomies or meta values, which is useful for e-commerce sites or blogs with complex content types.
  
- **Limiting Search Results to Specific Post Types**
  - This allows you to ensure that search results show only relevant content, like products or services, based on user queries.

### 5. **Best Practices and Tips**

- **Use Conditional Tags with Caution**
  - Ensure that you're using `is_main_query()` to target only the main query. This prevents unintended side effects on other custom queries.
  
- **Avoid Running on Admin Pages**
  - Use `!is_admin()` to prevent your changes from affecting admin area queries, which could interfere with dashboard functionality.
  
- **Performance Considerations**
  - Since this hook modifies the query, avoid using overly complex logic that could slow down query performance.

- **Debugging the Query**
  - Use the `WP_Query` debug functions, like `var_dump($query);`, within your `pre_get_posts` function to inspect how your modifications are affecting the query.

### 6. **Advanced Example: Filtering by Custom Taxonomy**

Here's an advanced example where you filter posts by a custom taxonomy named `genre` on the archive page:

```php
function filter_posts_by_genre( $query ) {
    if ( $query->is_main_query() && !is_admin() && is_archive() ) {
        // Check if the 'genre' taxonomy is set in the query
        if ( isset( $_GET['genre'] ) && !empty( $_GET['genre'] ) ) {
            $query->set( 'tax_query', array(
                array(
                    'taxonomy' => 'genre',
                    'field'    => 'slug',
                    'terms'    => sanitize_text_field( $_GET['genre'] ),
                ),
            ));
        }
    }
}
add_action( 'pre_get_posts', 'filter_posts_by_genre' );
```

This code checks if the `genre` query parameter is set and then filters the archive page to show only the posts belonging to that specific genre.

### Conclusion

The `pre_get_posts` hook is a versatile tool for customizing WordPress queries and tailoring content display across your site. It’s a must-know action for WordPress developers looking to create more dynamic and user-specific experiences. By leveraging this hook properly, you can significantly enhance how content is presented without affecting the core of WordPress or theme files.

---