## 📌  `single.php` & `page.php`

When working with WordPress, it's important to understand how the theme hierarchy works, especially when it comes to displaying individual posts and pages. The two files you're referencing—`single.php` and `page.php`—play crucial roles in determining how WordPress displays content.

### 1. **`single.php` for Individual Posts**
In WordPress, the `single.php` file is responsible for rendering individual posts. This is part of the **WordPress Template Hierarchy**, which decides which template file is used based on the type of content being displayed. Here’s how it works:

- **Purpose of `single.php`:**  
   The `single.php` template is used for displaying a single post in WordPress. When you click on an individual post from your blog or any archive page (like a category or tag archive), WordPress uses the `single.php` template to display the content of that post. It pulls the post's title, content, metadata (like the date and author), and formats them according to how the template is coded.

- **Customization of Posts:**  
   You can customize how each individual post looks by modifying the `single.php` file. This template may include the post title (`the_title()`), the post content (`the_content()`), comments section, post navigation, and other post-specific details like categories or tags.

- **Hierarchy & Specificity:**  
   WordPress allows even more specific post templates. If you want to create a different template for a specific post type, you can create a template like `single-{post_type}.php`. For example, if you have a custom post type called "news," you can create a `single-news.php` file to control the display of that specific post type. This allows greater flexibility in styling various kinds of posts.

**Fallbacks:**  
   - If there is no `single.php` file in your theme, WordPress will fall back to more general templates like `index.php` to display individual posts.

### 2. **`page.php` for Individual Pages**
In contrast, the `page.php` file is specifically used for displaying **static pages** in WordPress. Pages differ from posts because they are typically used for static content like "About Us," "Contact," or "Services" rather than chronological blog entries. Here's how `page.php` operates:

- **Purpose of `page.php`:**  
   The `page.php` file handles the layout for individual pages in WordPress. When you create a new page in WordPress (like an "About Us" page), WordPress will use the `page.php` template to render that page's content. This allows you to control the appearance of static pages separately from posts.

- **Customization of Pages:**  
   Like `single.php`, you can edit the `page.php` file to customize the layout of your pages. You might choose to display content differently for pages compared to posts. For instance, pages often don't include metadata like the author or publication date, which are more common in blog posts.

- **Hierarchy & Specificity:**  
   WordPress offers more granular control over how different pages are displayed:
   - If you want to create a unique layout for a specific page, you can create a custom page template. To do this, simply add a file like `page-{slug}.php` where `{slug}` is the page slug (e.g., `page-about.php` for the "About" page).
   - You can also create page templates by adding the following comment at the top of a PHP file:

     ```php
     <?php
     /*
     Template Name: Custom Template
     */
     ```

     This allows you to assign a specific template to a page directly from the WordPress dashboard when editing that page.

**Fallbacks:**  
   - If no `page.php` file is available, WordPress will default to `index.php` to display the page content.



### WordPress Template Hierarchy Overview
Both `single.php` and `page.php` are part of WordPress’s **Template Hierarchy**, a powerful system that defines how WordPress selects template files for various types of content. Here’s a quick breakdown:

- **For Single Posts:**
   1. `single-{post_type}.php` (for custom post types)
   2. `single.php` (for individual posts)
   3. `index.php` (fallback)

- **For Pages:**
   1. `page-{slug}.php` (for a specific page with a unique slug)
   2. `page.php` (for all pages)
   3. `index.php` (fallback)

This hierarchy allows you to build highly customizable layouts for different kinds of content across your WordPress site. By using `single.php` and `page.php` effectively, you ensure that posts and pages have the right look and functionality according to the needs of your theme.

---