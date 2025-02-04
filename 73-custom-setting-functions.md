Here’s a detailed breakdown of these WordPress functions, their purpose, core concepts, and practical use cases.

---

## 1. **add_menu_page()**
### **Purpose**:
This function adds a new top-level menu page to the WordPress admin dashboard.

### **Core Concepts**:
- Used for creating a custom admin page.
- Requires parameters like menu title, capability, menu slug, and callback function.
- Allows defining an icon and position in the menu.

### **Syntax**:
```php
add_menu_page( 
    string $page_title, 
    string $menu_title, 
    string $capability, 
    string $menu_slug, 
    callable $function = '', 
    string $icon_url = '', 
    int $position = null 
);
```

### **Use Case**:
Creating a custom admin panel for a plugin.

```php
function my_custom_menu_page() {
    add_menu_page(
        'My Custom Page',  // Page title
        'Custom Menu',      // Menu title
        'manage_options',   // Capability
        'custom-menu',      // Menu slug
        'my_custom_page_callback', // Callback function
        'dashicons-admin-generic', // Icon
        6                   // Position
    );
}

function my_custom_page_callback() {
    echo '<h1>Welcome to My Custom Page</h1>';
}

add_action('admin_menu', 'my_custom_menu_page');
```

---

## 2. **add_submenu_page()**
### **Purpose**:
Adds a submenu under an existing menu in the WordPress admin dashboard.

### **Core Concepts**:
- Extends an existing menu by adding more options.
- Useful for plugins requiring multiple settings pages.

### **Syntax**:
```php
add_submenu_page( 
    string $parent_slug, 
    string $page_title, 
    string $menu_title, 
    string $capability, 
    string $menu_slug, 
    callable $function = '' 
);
```

### **Use Case**:
Adding a submenu under "Settings".

```php
function my_custom_submenu_page() {
    add_submenu_page(
        'options-general.php', // Parent slug (Settings menu)
        'My Submenu Page',     // Page title
        'Custom Submenu',      // Menu title
        'manage_options',      // Capability
        'custom-submenu',      // Slug
        'my_submenu_callback'  // Callback function
    );
}

function my_submenu_callback() {
    echo '<h2>My Custom Submenu</h2>';
}

add_action('admin_menu', 'my_custom_submenu_page');
```

---

## 3. **add_options_page()**
### **Purpose**:
Adds a submenu under "Settings" for plugin settings.

### **Core Concepts**:
- Creates an entry inside "Settings".
- Used when registering options for a plugin.

### **Syntax**:
```php
add_options_page( 
    string $page_title, 
    string $menu_title, 
    string $capability, 
    string $menu_slug, 
    callable $function = '' 
);
```

### **Use Case**:
Adding a settings page for a custom plugin.

```php
function my_plugin_settings_page() {
    add_options_page(
        'My Plugin Settings', 
        'Plugin Settings', 
        'manage_options', 
        'my-plugin-settings', 
        'my_plugin_settings_callback'
    );
}

function my_plugin_settings_callback() {
    echo '<h1>My Plugin Settings</h1>';
}

add_action('admin_menu', 'my_plugin_settings_page');
```

---

## 4. **add_settings_section()**
### **Purpose**:
Creates a section inside a settings page.

### **Core Concepts**:
- Organizes settings into sections.
- Used with `add_settings_field()`.

### **Syntax**:
```php
add_settings_section( 
    string $id, 
    string $title, 
    callable $callback, 
    string $page 
);
```

### **Use Case**:
Adding a section to the settings page.

```php
function my_plugin_register_settings() {
    add_settings_section(
        'my_plugin_section', 
        'My Plugin Settings', 
        'my_plugin_section_callback', 
        'my-plugin-settings'
    );
}

function my_plugin_section_callback() {
    echo '<p>Configure your plugin here.</p>';
}

add_action('admin_init', 'my_plugin_register_settings');
```

---

## 5. **add_settings_field()**
### **Purpose**:
Adds a setting field to a section.

### **Core Concepts**:
- Used with `register_setting()`.
- Defines input fields like text, dropdown, checkbox.

### **Syntax**:
```php
add_settings_field( 
    string $id, 
    string $title, 
    callable $callback, 
    string $page, 
    string $section = 'default', 
    array $args = []
);
```

### **Use Case**:
Adding a text input field to a settings page.

```php
function my_plugin_register_fields() {
    add_settings_field(
        'my_plugin_option', 
        'Enter a Value', 
        'my_plugin_option_callback', 
        'my-plugin-settings', 
        'my_plugin_section'
    );

    register_setting('my-plugin-settings', 'my_plugin_option');
}

function my_plugin_option_callback() {
    $value = get_option('my_plugin_option', '');
    echo '<input type="text" name="my_plugin_option" value="' . esc_attr($value) . '">';
}

add_action('admin_init', 'my_plugin_register_fields');
```

---

## 6. **settings_fields()**
### **Purpose**:
Outputs hidden fields for settings security.

### **Core Concepts**:
- Used inside forms in the settings page.
- Works with `register_setting()`.

### **Syntax**:
```php
settings_fields( string $option_group );
```

### **Use Case**:
Adding a form to save settings.

```php
function my_plugin_settings_page_callback() {
    echo '<form method="post" action="options.php">';
    settings_fields('my-plugin-settings');
    do_settings_sections('my-plugin-settings');
    submit_button();
    echo '</form>';
}
```

---

## 7. **do_settings_sections()**
### **Purpose**:
Displays all sections and fields on a settings page.

### **Core Concepts**:
- Works with `add_settings_section()` and `add_settings_field()`.

### **Syntax**:
```php
do_settings_sections( string $page );
```

### **Use Case**:
Automatically displaying fields inside a settings page.

```php
function my_plugin_settings_page_callback() {
    echo '<h1>Plugin Settings</h1>';
    echo '<form method="post" action="options.php">';
    settings_fields('my-plugin-settings');
    do_settings_sections('my-plugin-settings');
    submit_button();
    echo '</form>';
}
```

---

## 8. **submit_button()**
### **Purpose**:
Displays a submit button.

### **Core Concepts**:
- Simplifies adding a button inside settings forms.

### **Syntax**:
```php
submit_button( string $text = 'Save Changes', string $type = 'primary', string $name = 'submit', bool $wrap = true, array $other_attributes = [] );
```

### **Use Case**:
Adding a button inside a settings form.

```php
submit_button('Save Changes');
```

---

## 9. **add_settings_error()**
### **Purpose**:
Displays validation errors for settings.

### **Core Concepts**:
- Works with `settings_errors()`.
- Used when validating settings.

### **Syntax**:
```php
add_settings_error( 
    string $setting, 
    string $code, 
    string $message, 
    string $type = 'error' 
);
```

### **Use Case**:
Validating input before saving.

```php
function my_plugin_validate_options($input) {
    if (empty($input)) {
        add_settings_error('my_plugin_option', 'empty_field', 'Field cannot be empty!', 'error');
    }
    return $input;
}
```

---

## **Final Thoughts**
These functions are crucial for creating custom settings pages in WordPress. They help in organizing settings, adding options, and managing admin pages efficiently.

Would you like a full implementation of a settings page using these functions? 🚀