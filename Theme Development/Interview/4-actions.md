## 📌 **Actions in WordPress**

**Actions** in WordPress are a fundamental part of the WordPress **hook system**, allowing developers to execute custom code at specific points during the WordPress lifecycle. By using actions, you can tap into various points of WordPress’s process (e.g., when a post is published, a theme is loaded, or a comment is posted) to run your own functions.

### **Purpose of Actions**

The primary purpose of **actions** is to allow developers to execute specific tasks at predefined stages of WordPress execution. They are a type of **hook** that signals WordPress to perform certain tasks, whether modifying behavior, extending functionality, or interacting with plugins, themes, and core WordPress components.

**For example**, you might want to run custom code whenever a new post is published, or enqueue a custom stylesheet when a theme is loaded. Actions provide that control, enabling you to integrate custom functions at various key points in the WordPress workflow.

### **Core Concepts of Actions**

1. **Hooks:**
   Actions are a type of **hook** in WordPress, and they allow you to hook custom functionality at certain points within WordPress’s execution.
   
2. **Action Hooks vs. Filter Hooks:**
   - **Action Hooks**: Actions **execute custom code** at specific points in the execution of WordPress (like saving a post, rendering a template, or sending an email).
   - **Filter Hooks**: Filters **modify data** at specific points but must return the modified data.

3. **Action Hooks Are Do-Not-Return:**
   Unlike filters, action hooks don’t modify data and don’t require a return value. They are designed purely for side-effects (executing custom code like logging, sending emails, or displaying custom messages).

4. **add_action() Function:**
   This is the main function used to tie a custom function to an action hook in WordPress. It allows you to "attach" your function to a specific action hook in WordPress.
   ```php
   add_action( 'hook_name', 'your_function_name', [priority], [accepted_args] );
   ```
   - **hook_name**: The name of the action you want to hook into (e.g., `'wp_footer'`).
   - **your_function_name**: The custom function you want to execute when the action is triggered.
   - **priority** (optional): The order in which to execute this function in relation to other functions hooked to the same action (default is 10).
   - **accepted_args** (optional): The number of arguments the function accepts (default is 1).

5. **do_action() Function:**
   This function is used within the core WordPress code, plugins, or themes to create custom actions. It triggers the execution of all functions attached to that action.
   ```php
   do_action( 'hook_name', [arg1], [arg2], ... );
   ```
   - **hook_name**: The action hook to which functions are attached.
   - **arg1, arg2, ...**: Additional arguments that can be passed to the functions hooked to the action.

### **Common Built-In Action Hooks in WordPress**

WordPress offers many **built-in action hooks** that developers can use to customize or extend the platform's behavior. Here are some commonly used ones:

1. **`wp_enqueue_scripts`**:
   Triggered when stylesheets and scripts are being enqueued. This is commonly used to load custom CSS or JS files in a theme or plugin.
   ```php
   function enqueue_custom_scripts() {
       wp_enqueue_style( 'custom-style', get_stylesheet_uri() );
       wp_enqueue_script( 'custom-js', get_template_directory_uri() . '/custom.js', array(), null, true );
   }
   add_action( 'wp_enqueue_scripts', 'enqueue_custom_scripts' );
   ```

2. **`init`**:
   Triggered after WordPress has finished loading but before any headers are sent. This hook is often used to register custom post types, taxonomies, or initialize features.
   ```php
   function custom_init() {
       // Custom post type or other initialization
   }
   add_action( 'init', 'custom_init' );
   ```

3. **`wp_head`**:
   Triggered within the `<head>` section of the HTML document. Often used to insert meta tags, custom styles, or JavaScript.
   ```php
   function add_custom_meta() {
       echo '<meta name="custom-meta" content="WordPress actions">';
   }
   add_action( 'wp_head', 'add_custom_meta' );
   ```

4. **`save_post`**:
   Triggered when a post is created or updated in WordPress. This is useful when you need to perform an action after a post is saved, such as updating metadata or sending notifications.
   ```php
   function post_save_action( $post_id ) {
       if ( get_post_type( $post_id ) == 'post' ) {
           // Perform an action when a post is saved
       }
   }
   add_action( 'save_post', 'post_save_action' );
   ```

5. **`wp_footer`**:
   Triggered just before the closing `</body>` tag. It is often used to insert custom scripts or tracking codes, such as Google Analytics.
   ```php
   function add_footer_script() {
       echo '<script src="custom-script.js"></script>';
   }
   add_action( 'wp_footer', 'add_footer_script' );
   ```

6. **`admin_menu`**:
   Triggered when the WordPress admin dashboard menu is being created. It is often used to add custom admin pages.
   ```php
   function add_custom_menu_item() {
       add_menu_page( 'Custom Page', 'Custom Menu', 'manage_options', 'custompage', 'custom_page_callback' );
   }
   add_action( 'admin_menu', 'add_custom_menu_item' );
   ```

7. **`template_redirect`**:
   Triggered before the WordPress template is rendered, allowing you to perform redirects or load custom templates before content is displayed.
   ```php
   function redirect_non_logged_in_users() {
       if ( !is_user_logged_in() && is_page( 'members-only' ) ) {
           wp_redirect( home_url( '/login' ) );
           exit;
       }
   }
   add_action( 'template_redirect', 'redirect_non_logged_in_users' );
   ```

### **Custom Actions with `do_action()`**

You can also create your own custom action hooks within themes or plugins using the `do_action()` function. This allows others to hook into your custom action to add their own functionality.

#### Example: Custom Action Hook
```php
function my_custom_action_function() {
    // Custom code to be executed
}
add_action( 'my_custom_hook', 'my_custom_action_function' );

// Triggering the action
do_action( 'my_custom_hook' );
```

In this example:
- **`add_action()`** is used to register a function to the custom hook `my_custom_hook`.
- **`do_action()`** is then called to trigger the custom action, which in turn executes all functions attached to `my_custom_hook`.

### **Use Cases for Actions in Practice**

1. **Enqueuing Styles and Scripts**
   You can use the `wp_enqueue_scripts` action to load custom stylesheets and scripts.
   ```php
   function load_custom_scripts() {
       wp_enqueue_style( 'custom-style', get_stylesheet_uri() );
   }
   add_action( 'wp_enqueue_scripts', 'load_custom_scripts' );
   ```

2. **Sending Notifications on Post Publish**
   Using the `publish_post` action, you can send notifications to users when a new post is published.
   ```php
   function notify_user_on_post_publish( $post_id ) {
       // Send an email when a post is published
       wp_mail( 'email@example.com', 'New Post Published', 'A new post has been published on the site.' );
   }
   add_action( 'publish_post', 'notify_user_on_post_publish' );
   ```

3. **Adding Custom Code in the Footer**
   You can add a script or tracking code like Google Analytics in the footer using the `wp_footer` action.
   ```php
   function add_tracking_code() {
       echo '<script>/* Tracking code here */</script>';
   }
   add_action( 'wp_footer', 'add_tracking_code' );
   ```

4. **Custom Post Save Actions**
   When a post is saved, you can perform additional tasks like updating metadata or clearing cache.
   ```php
   function custom_save_post( $post_id ) {
       if ( 'post' == get_post_type( $post_id ) ) {
           // Update metadata or perform custom tasks
       }
   }
   add_action( 'save_post', 'custom_save_post' );
   ```

5. **Redirecting Users Based on Conditions**
   Use the `template_redirect` action to redirect users to a login page if they are not logged in.
   ```php
   function redirect_if_not_logged_in() {
       if ( !is_user_logged_in() && !is_page( 'login' ) ) {
           wp_redirect( home_url( '/login' ) );
           exit;
       }
   }
   add_action( 'template_redirect', 'redirect_if_not_logged_in' );
   ```

### **Key Takeaways:**
- **Actions** allow you to add custom functionality at specific points during WordPress’s execution.
- **add_action()** attaches your function to an action hook, while **do_action()** triggers the execution of that action.
- WordPress provides a wide range of **built-in action hooks** that can be used for customizing themes, plugins, and core functionality