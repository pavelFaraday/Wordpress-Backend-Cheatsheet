# 📌 Custom Post Types (CPT)

**In WordPress, a Custom Post Type (CPT) allows you to create and manage different types of content beyond the default types like Posts and Pages. Essentially, it's a way to extend WordPress’s functionality to handle content more suited to your needs**.

#### Default Post Types in WordPress

By default, WordPress comes with a few built-in post types:

- **Posts**: Used for blog entries.
- **Pages**: Used for static content (e.g., About page).
- **Attachments**: Handles media uploads.
- **Revisions**: Tracks changes made to content.
- **Menus**: Used for navigation menus.


  While these default types work for most cases, there are situations where you need more specific types of content, and that’s where CPTs come in.

#### What Is a Custom Post Type?

**A Custom Post Type is a structure you can create to display unique types of content.** For example, if you’re building a movie review website, you might want a "Movies" CPT where each entry contains a movie’s title, release date, and director. Similarly, a real estate site might have a "Properties" CPT with fields for location, price, and number of rooms.

#### Why Use Custom Post Types?

- **Organization**: Helps keep different content types separate, making your site more organized.
- **Better Functionality**: Each CPT can have its own custom fields, templates, and features.
- **Tailored Admin Panel**: You can create custom taxonomies (like categories and tags) and customize the WordPress admin interface for specific content.

#### Creating a Custom Post Type

You can create a CPT by adding custom code to your theme’s `functions.php` file or using a plugin like Custom Post Type UI. Here’s a simple code example to register a "Movies" custom post type:

```php
function university_post_types() {
    register_post_type('events',
        array(
            'public' => true,
            'show_in_rest' => true,
            'menu_icon' => 'dashicons-calendar-alt',
            'has_archive' => true,
            'rewrite' => array('slug' => 'Events'),
            'supports' => array('title', 'editor', 'thumbnail', 'custom-fields'),
            'labels' => array (
                'name' => __('Events'),
                'add_new_item' => 'Add New Event',
                'edit_item' => 'Edit Event',
                'all_items' => 'All Events',
                'singular_name' => __('Event')
            )
        )
    );
  }
  add_action('init', 'university_post_types');
```

#### Use Cases for Custom Post Types

- Portfolio: Showcasing individual projects or case studies.
- Testimonials: Displaying customer reviews or feedback.
- Products: Creating an e-commerce catalog.
- Events: Managing an event calendar.

**Using Custom Post Types allows you to make WordPress even more flexible, adapting it perfectly to different types of content for custom websites.**

---
---
---