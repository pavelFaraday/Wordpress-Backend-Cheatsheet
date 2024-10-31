## 📌  Actions, Hooks & Filters in WordPress

In WordPress development, **Actions**, **Hooks**, and **Filters** are core elements that allow you to extend and modify the behavior of WordPress without directly altering its core files. Let’s break down these concepts in detail:

---

### **1. Actions, Hooks & Filters - Overview**

> - **Actions** and **Filters** are both types of **Hooks**. Hooks allow developers to *“hook into”* or customize WordPress functionality.
> - **Actions** are used to **perform a task or execute code at specific points.**
> - **Filters** are used to **modify data** before it is sent to the **database** or **browser**.

---

### **2. Purpose**

- **Actions**: To execute a specific code or function at a particular point in WordPress, like when a post is published, an admin page is loaded, or a comment is saved. 
- **Filters**: **To modify or filter content/data**, such as changing the title of a post or adding custom data to a user’s profile information.
- **Hooks**: **The <u>umbrella</u> term for both Actions and Filters. It signifies predefined points in WordPress where custom code can be attached.**

---

### **3. Core Concepts**

#### **A. Actions**
- **Definition**: Actions are hooks that allow you to insert custom code at specific points in the WordPress execution cycle.
- **Mechanism**: **When an action is triggered, all functions attached to it are executed in order of priority.**
- **Usage**: You use actions to **perform a specific task that does not modify any specific data**, such as:
  - Sending an email when a post is published.
  - Adding custom widgets or sidebars.
  - Enqueueing scripts or stylesheets.

**Syntax**:
- **Adding an Action**: `add_action('action_name', 'callback_function', [priority], [accepted_args]);`
- **Executing an Action**: `do_action('action_name', [parameters]);`

**Example**:
```php
// Registering an action to execute a custom function after a post is published.
add_action('publish_post', 'notify_author_on_publish', 10, 2);

function notify_author_on_publish($post_ID, $post) {
    $author_email = get_the_author_meta('user_email', $post->post_author);
    wp_mail($author_email, "Your post has been published!", "Congratulations!");
}
```

#### **B. Filters**
- **Definition**: **Filters are hooks that modify data before it is displayed or stored in the database.**
- **Mechanism**: **When a filter is applied, all functions attached to it process the data and return it for further use.**
- **Usage**: You use filters to modify content like:
  - Altering post titles or content before rendering.
  - Adding custom data to an array.
  - Changing text in email notifications or custom fields.

**Syntax**:
- **Adding a Filter**: `add_filter('filter_name', 'callback_function', [priority], [accepted_args]);`
- **Applying a Filter**: `apply_filters('filter_name', $data, [parameters]);`

**Example**:
```php
// Adding a filter to change the title of a post.
add_filter('the_title', 'modify_post_title', 10, 1);

function modify_post_title($title) {
    return '🔔 ' . $title; // Adding an emoji before the title
}
```

#### **C. Hooks**
- **Definition**: Hooks are the foundation that allows Actions and Filters to exist. They are predefined points within the WordPress core.
- **Purpose**: **Hooks act as breakpoints or triggers within WordPress, providing opportunities to <u>add or modify functionality</u>.**
- **Types**:
  > - **Action Hooks**: Triggers specific tasks.
  > - **Filter Hooks**: Modifies and returns data.

**Common Hooks**:
- `init`: Runs after WordPress has finished loading but before any headers are sent.
- `wp_head`: Injects content in the `<head>` section.
- `the_content`: Filters post content.
- `save_post`: Executes when a post is saved.

---

### **4. Use Cases in Practice**

#### **Use Case 1: Sending an Email when a Post is Published**
```php
function custom_send_email_on_publish($post_ID, $post) {
    $author_email = get_the_author_meta('user_email', $post->post_author);
    $post_title = get_the_title($post_ID);
    wp_mail($author_email, "Your post '{$post_title}' is live!", "Congrats on publishing a new post!");
}
add_action('publish_post', 'custom_send_email_on_publish', 10, 2);
```
In this example:
- The action `publish_post` is used to trigger an email when a post is published.

#### **Use Case 2: Modifying the Post Content**
```php
function append_custom_text_to_content($content) {
    if (is_single()) {
        $custom_text = "<p>Thanks for reading! Stay tuned for more updates.</p>";
        $content .= $custom_text;
    }
    return $content;
}
add_filter('the_content', 'append_custom_text_to_content');
```
In this example:
- The filter `the_content` is used to append a message at the end of every post.

#### **Use Case 3: Registering a Custom Sidebar**
```php
function register_custom_sidebar() {
    register_sidebar(array(
        'name'          => 'Custom Sidebar',
        'id'            => 'custom_sidebar',
        'before_widget' => '<div class="custom-widget">',
        'after_widget'  => '</div>',
        'before_title'  => '<h3 class="custom-widget-title">',
        'after_title'   => '</h3>',
    ));
}
add_action('widgets_init', 'register_custom_sidebar');
```
In this example:
- The action `widgets_init` is used to register a custom sidebar in WordPress.

---

### **5. Best Practices**

- **Use descriptive names** for custom hooks and callbacks to maintain code clarity.
- **Manage priorities wisely**. If you want a particular function to execute first or last, use the priority parameter.
- **Return data in Filters**. <u>**Always remember to return modified data in filter functions.</u>**
- **Don’t directly edit core files**. Instead, rely on actions and filters to make modifications.

---

### **6. Summary**

- **Actions**: Execute custom code at specific points in WordPress. They’re like saying *“Do this”* when something happens.
- **Filters**: Modify data before using it. They’re like saying *“Change this”* to certain pieces of content or data.
- **Hooks**: Predefined places in WordPress that allow you to attach your actions or filters.

Understanding and using actions, hooks, and filters effectively is key to developing flexible, powerful, and maintainable WordPress websites. **They enable developers to add custom features and alter default behavior without altering core WordPress files—a crucial aspect of WordPress best practices.**

---

Let’s expand on Actions, Filters, and Hooks with additional insights and use cases to better understand how they fit into the WordPress ecosystem.

### **1. Actions in WordPress**

#### **More Details**
Actions are fired at specific points during WordPress’ execution cycle. They allow you to run custom code when a particular WordPress event or trigger occurs. WordPress has a vast number of built-in action hooks, but you can also create your own custom action hooks.

#### **Action Hook Execution Sequence**
**When an action hook is called, all the functions associated with that hook are executed sequentially based on their priority (default is 10). If two functions have the same priority, they execute in the order they were added.**

#### **More Common Action Hooks**
- `init`: Runs after WordPress is loaded and is commonly used to initialize plugins or custom post types.
- `admin_init`: Fires when the admin panel is initialized.
- `wp_enqueue_scripts`: Ideal for enqueuing scripts and styles on the front end.
- `save_post`: Executes when a post is saved, updated, or published.
- `wp_logout`: Executes when a user logs out of the website.
- `user_register`: Executes when a new user registers.

#### **Additional Use Cases for Actions**

1. **Enqueueing Custom Scripts and Styles**:
   ```php
   function enqueue_custom_scripts() {
       wp_enqueue_style('custom-style', get_template_directory_uri() . '/css/custom-style.css');
       wp_enqueue_script('custom-script', get_template_directory_uri() . '/js/custom-script.js', array('jquery'), null, true);
   }
   add_action('wp_enqueue_scripts', 'enqueue_custom_scripts');
   ```
   **Purpose**: Uses the `wp_enqueue_scripts` action hook to include custom CSS and JavaScript files on your WordPress site.

2. **Creating Custom User Meta Data upon Registration**:
   ```php
   function add_custom_user_meta($user_id) {
       add_user_meta($user_id, 'user_rank', 'newbie');
   }
   add_action('user_register', 'add_custom_user_meta');
   ```
   **Purpose**: This example uses the `user_register` action hook to add custom meta data for a newly registered user.

3. **Redirecting After Login**:
   ```php
   function custom_login_redirect($redirect_to, $requested_redirect_to, $user) {
       if (isset($user->roles) && is_array($user->roles)) {
           if (in_array('administrator', $user->roles)) {
               return admin_url(); // Redirect admins to the dashboard
           } else {
               return home_url(); // Redirect all other users to the homepage
           }
       }
       return $redirect_to;
   }
   add_filter('login_redirect', 'custom_login_redirect', 10, 3);
   ```
   **Purpose**: Uses a filter (`login_redirect`) to conditionally redirect users after they log in based on their roles.

---

### **2. Filters in WordPress**

#### **More Details**
Filters are powerful tools in WordPress that allow you to modify and return data. Unlike actions, filters must return the modified data back to WordPress after they are processed.

#### **Filter Execution Sequence**
**Filters execute attached functions based on their priority. Each attached function must return a modified or unmodified value.**

#### **Common Filters**
- `the_content`: Modifies post content before displaying it.
- `the_title`: Filters the post or page title.
- `excerpt_length`: Changes the default length of post excerpts.
- `login_redirect`: Customizes where users are redirected after login.
- `widget_title`: Modifies the titles of sidebar widgets.
- `wp_mail_from`: Filters the default “from” email address for outgoing WordPress emails.

#### **Additional Use Cases for Filters**

1. **Changing the Length of the Excerpt**:
   ```php
   function custom_excerpt_length($length) {
       return 20; // Changing the length of the excerpt to 20 words
   }
   add_filter('excerpt_length', 'custom_excerpt_length');
   ```
   **Purpose**: Uses the `excerpt_length` filter to limit the length of the post excerpts.

2. **Adding a Custom Class to Body**:
   ```php
   function add_custom_body_class($classes) {
       if (is_single()) {
           $classes[] = 'single-post-custom-class';
       }
       return $classes;
   }
   add_filter('body_class', 'add_custom_body_class');
   ```
   **Purpose**: Uses the `body_class` filter to append a custom class to the `<body>` tag based on the page type.

3. **Modifying the Email “From” Name**:
   ```php
   function custom_mail_from_name($name) {
       return 'My Custom Name';
   }
   add_filter('wp_mail_from_name', 'custom_mail_from_name');
   ```
   **Purpose**: Changes the default “from” name in outgoing WordPress emails using the `wp_mail_from_name` filter.

---

### **3. Hooks in WordPress**

#### **More Details**
**Hooks are <u>*strategically placed points in WordPress code where developers can insert custom functions*</u>. You can use built-in hooks or create your own.**

#### **Creating Custom Hooks**

Sometimes you want to allow other developers to hook into your custom plugin or theme. You can create your custom action or filter hooks using the `do_action()` or `apply_filters()` functions.

**Creating a Custom Action Hook**:
```php
// Creating a custom action hook in a plugin or theme
do_action('my_custom_action', $some_data);

// Adding a function to execute when the custom action is called
add_action('my_custom_action', 'handle_custom_action');

function handle_custom_action($data) {
    // Perform some action with $data
}
```

**Creating a Custom Filter Hook**:
```php
// Creating a custom filter hook in a plugin or theme
$filtered_data = apply_filters('my_custom_filter', $original_data);

// Adding a function to modify the data when the custom filter is called
add_filter('my_custom_filter', 'modify_custom_data');

function modify_custom_data($data) {
    // Modify and return the data
    return $data . ' - Modified';
}
```

#### **Common Scenarios for Custom Hooks**

1. **Allowing Third-Party Developers to Extend Plugins or Themes**:
   - If you’re developing a theme or plugin, you can place custom hooks to allow other developers to add or modify functionality.

2. **Creating Custom Payment Processing**:
   - In an e-commerce plugin, you can create custom action hooks like `do_action('custom_payment_processed', $order_id)` to allow other developers to attach their code to payment events.

3. **Custom Content Rendering**:
   - In a theme’s template, you can add a custom hook like `do_action('after_single_post')` to let other developers inject custom content or features after a single post.

---

### **4. Actions vs Filters - Key Differences**

- **Execution**:
  - **Actions** <u>execute functions to perform a task.</u>
  - **Filters** execute functions to <u>modify and return data.</u>
- **Return Value**:
  - **Actions** **do not return any data** (though functions within them can).
  - **Filters** **<u>must</u> return modified data.**
- **Purpose**:
  - **Actions** <u>focus on triggering additional functions.</u>
  - **Filters** <u>focus on modifying specific data.</u>

---

### **5. Summary of Additional Use Cases**

1. **Using Action Hooks for Custom Dashboard Widgets**:
   ```php
   function custom_dashboard_widget() {
       wp_add_dashboard_widget(
           'custom_widget_id',
           'My Custom Widget',
           'custom_widget_display_function'
       );
   }
   add_action('wp_dashboard_setup', 'custom_dashboard_widget');

   function custom_widget_display_function() {
       echo "Welcome to my custom dashboard widget!";
   }
   ```
   **Purpose**: Adds a custom widget to the WordPress admin dashboard using the `wp_dashboard_setup` action.

2. **Using Filter Hooks to Customize the Admin Footer Text**:
   ```php
   function custom_admin_footer_text() {
       return "Thank you for creating with WordPress. Powered by Your Company.";
   }
   add_filter('admin_footer_text', 'custom_admin_footer_text');
   ```
   **Purpose**: Modifies the default footer text in the WordPress admin area using the `admin_footer_text` filter.

3. **Applying Custom Logic with Hooks in WooCommerce**:
   ```php
   function add_gift_to_cart() {
       if (is_user_logged_in()) {
           WC()->cart->add_to_cart('1234'); // Add a product to the cart if the user is logged in
       }
   }
   add_action('woocommerce_before_cart', 'add_gift_to_cart');
   ```
   **Purpose**: Uses WooCommerce’s action hook `woocommerce_before_cart` to add a free gift to the cart for logged-in users.

---

By mastering the use of Actions, Filters, and Hooks in WordPress, you gain the ability to make extensive customizations without editing core files or violating best practices. This flexibility is what makes WordPress an incredibly powerful and adaptable platform for all levels of developers.