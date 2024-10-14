### WordPress Theme Functions: A Comprehensive Overview

Here are short descriptions and the differences between the WordPress functions you've listed:

To make it clear and organized, you can categorize these functions into specific groups based on their purpose. Here's a headline structure that you could use:

#### 1. **Main Stylesheet URL Retrieval**
   - `get_stylesheet_uri()`

#### 2. **Theme File URL Retrieval**
   - `get_theme_file_uri()`
   - `get_stylesheet_directory_uri()`
   - `get_template_directory_uri()`
   - `get_theme_root_uri()`

#### 3. **Theme File Path Retrieval**
   - `get_theme_file_path()`
   - `get_stylesheet_directory()`
   - `get_template_directory()`
   - `get_theme_root()`

#### 4. **Template File Handling**
   - `get_template_part()`
   - `locate_template()`

This categorization highlights each function's role in URL retrieval, file path retrieval, and template file handling, making it easier to understand their differences and use cases.

---

### 1. `get_stylesheet_uri()`
- **Description**: Returns the URI (URL) of the main stylesheet (`style.css`) of the current active theme.
- **Use Case**: Use it to link directly to the main stylesheet of your child theme or active theme.
- **Example**: Outputs something like `https://example.com/wp-content/themes/your-theme/style.css`.

### 2. `get_theme_file_uri()`
- **Description**: Returns the URI (URL) of a file in the current active theme or child theme.
- **Use Case**: Use it when you need the URL of any file (not just CSS) in the theme's folder.
- **Example**: `get_theme_file_uri('assets/js/script.js')` would output the URL to `script.js` in the theme.

### 3. `get_template_directory_uri()`
- **Description**: Returns the URI (URL) of the parent theme directory.
- **Use Case**: Use this to reference files in the parent theme even when using a child theme.
- **Example**: Outputs something like `https://example.com/wp-content/themes/parent-theme`.

### 4. `get_theme_file_path()`
- **Description**: Returns the absolute file path of a file in the current theme or child theme.
- **Use Case**: Use it to retrieve server paths for files within the theme directory.
- **Example**: `get_theme_file_path('template-parts/header.php')` would output the file path to `header.php` in the theme.

### 5. `get_stylesheet_directory()`
- **Description**: Returns the absolute file path to the directory of the active theme, including child themes.
- **Use Case**: Use this when you need the server path to the current active theme, such as for child theme customization.
- **Example**: Outputs something like `/var/www/html/wp-content/themes/child-theme`.

### 6. `get_template_directory()`
- **Description**: Returns the absolute file path to the parent theme directory.
- **Use Case**: Use this to reference files directly in the parent theme, even if a child theme is active.
- **Example**: Outputs something like `/var/www/html/wp-content/themes/parent-theme`.

### Differences
- **Parent vs. Child Theme**:
  - `get_stylesheet_uri()` and `get_stylesheet_directory()` always point to the child theme (if one is active).
  - `get_template_directory_uri()` and `get_template_directory()` always point to the parent theme.

- **URI vs. File Path**:
  - Functions with `URI` in the name (e.g., `get_stylesheet_uri()`, `get_theme_file_uri()`) return URLs.
  - Functions with `Path` or `Directory` in the name (e.g., `get_theme_file_path()`, `get_template_directory()`) return server file paths.

- **Specific File Retrieval**:
  - `get_theme_file_uri()` and `get_theme_file_path()` are versatile as they allow specifying a file within the theme.

---

### Additional Functions

1. **`get_stylesheet_directory_uri()`**
   - **Description**: Returns the URL of the directory of the current active theme (child theme if one is being used).
   - **Use Case**: Use it when you need to get the URL to the root directory of the active theme.
   - **Difference**: It points to the child theme directory when a child theme is active.

2. **`get_template_part()`**
   - **Description**: Includes a template part file from the current theme or child theme.
   - **Use Case**: Use it to modularize your theme code by including template parts like headers, footers, or sidebars.
   - **Example**: `get_template_part('template-parts/content', 'single')` includes a file named `content-single.php` from the `template-parts` directory.

3. **`locate_template()`**
   - **Description**: Looks for a template file in the child theme first and then in the parent theme if the file is not found in the child.
   - **Use Case**: Use it to ensure you load the appropriate file from either the child or parent theme.
   - **Example**: `locate_template(array('custom-template.php'), true)` will look for the file in both the child and parent theme.

4. **`get_theme_root()`**
   - **Description**: Returns the absolute path to the themes directory.
   - **Use Case**: Useful when you need the base path where all themes are stored on the server.
   - **Example**: Outputs something like `/var/www/html/wp-content/themes`.

5. **`get_theme_root_uri()`**
   - **Description**: Returns the URL to the root directory where all themes are stored.
   - **Use Case**: Use it when you need the base URL for the themes directory.
   - **Example**: Outputs something like `https://example.com/wp-content/themes`.

### Summary of Related Functions
Here's a quick overview to help you understand their specific purposes:

- **Current Theme URL (including child themes)**: 
  - `get_stylesheet_uri()` – Main stylesheet URL of the active theme.
  - `get_stylesheet_directory_uri()` – URL to the active theme directory.

- **Parent Theme URL**:
  - `get_template_directory_uri()` – URL to the parent theme directory.

- **Paths on the Server**:
  - `get_stylesheet_directory()` – File path to the active theme directory.
  - `get_template_directory()` – File path to the parent theme directory.
  - `get_theme_file_path()` – File path to a specific file in the current theme.

- **Finding Files in Themes**:
  - `locate_template()` – Finds and loads files from child or parent theme.
  - `get_template_part()` – Includes template part files from the theme.

These functions help manage how you load files, whether you are working with child themes or directly with the parent theme.