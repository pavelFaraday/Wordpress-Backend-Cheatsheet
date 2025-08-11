# 📌 WordPress Loops

- [📌 WordPress Loops](#-wordpress-loops)
    - [Overview — what the Loop _is_ and _why_ it matters](#overview--what-the-loop-is-and-why-it-matters)
    - [Interview-ready cheat sheet (quick answers you can say)](#interview-ready-cheat-sheet-quick-answers-you-can-say)
    - [Managing relationships between CPTs (Game ↔ Casino)](#managing-relationships-between-cpts-game--casino)
    - [Core concepts \& functions (the essentials)](#core-concepts--functions-the-essentials)
    - [Common mistakes to avoid (say these in interview to show depth)](#common-mistakes-to-avoid-say-these-in-interview-to-show-depth)
- [Loops](#loops)
    - [1. Default Loop - (`have_posts()`)](#1-default-loop---have_posts)
    - [2. Custom Query Loop (`WP_Query`)](#2-custom-query-loop-wp_query)
    - [3. Secondary Loop (Nested Loops)](#3-secondary-loop-nested-loops)
    - [4. `query_posts()` Loop](#4-query_posts-loop)
    - [5. `get_posts()` Loop](#5-get_posts-loop)
    - [6. Taxonomy Loop](#6-taxonomy-loop)
    - [7. Custom Post Type (CPT) Loop](#7-custom-post-type-cpt-loop)
    - [8. Meta Query Loop](#8-meta-query-loop)
    - [9. Date Query Loop](#9-date-query-loop)
    - [10. Author Loop](#10-author-loop)
    - [11. Search Results Loop](#11-search-results-loop)
    - [12. Pagination Loop](#12-pagination-loop)
---

  - **Default Loop - `have_posts()`**
> *The main loop that displays posts on blog index, archives, categories, or any default WordPress query.*
  - **Custom Query Loop - `WP_Query`**
> *Runs custom database queries without interfering with the main loop, allowing for advanced filtering.*
  - **Secondary Loop (Nested Loops)**
> *Runs multiple queries on the same page without disturbing the main content.*
  - `query_posts()` Loop
> *Modifies the main query but generally. Therefor, We should **AVOID** it❗️*
  - **`get_posts()` Loop**
> *Quickly fetch a simple array of posts; Small post lists in widgets or custom blocks*
  - **Taxonomy Loop**
> *Retrieves category lists, tags, or custom taxonomy filters.*
  - **CPT Loop**
>*Display posts from custom post types: portfolios, events, products, Casino Games or other CPT content*
  - **Meta Query Loop**
  > Filter posts by custom fields/meta data: by price, events by date, posts marked as featured
  - **Date Query Loop**
  > Filter posts by date: Archive pages by month, year or custom date ranges
  - **Author Loop**
  > Show posts by specific author: Author archive pages,  Team member portfolios
  - **Search Results Loop**
  > Show results matching user search queries: Search results page
  - **Pagination Loop**
  > Divide post listings into pages for better UX.

---

### Overview — what the Loop _is_ and _why_ it matters

The **Loop** is WordPress’s central pattern for iterating over a set of posts returned by a query (the _main query_ or any _custom query_). It’s the glue between querying data (`WP_Query`) and rendering templates (`the_title()`, `the_content()`, etc.). Every theme template that outputs multiple posts (index, archive, search, custom page templates) uses a Loop.

---

### Interview-ready cheat sheet (quick answers you can say)

- **Q: What is the Loop?**
  A: The Loop is WordPress’s pattern for iterating over posts returned by a `WP_Query` — `have_posts()` + `the_post()` set up template data for template tags.

- **Q: Difference: `WP_Query` vs `get_posts` vs `query_posts`?**
  A: `WP_Query` is flexible and used for secondary loops. `get_posts()` returns an array and is lighter. `query_posts()` modifies the main query and is discouraged.

- **Q: How to safely run two loops on a page?**
  A: Use `new WP_Query()` for the secondary loop and call `wp_reset_postdata()` afterwards.

- **Q: Why use `wp_reset_postdata()`?**
  A: It restores the global `$post` to the main query’s state so subsequent template tags work correctly.

- **Q: How to add CPTs to blog home?**
  A: Use `pre_get_posts` and modify the main query; check `is_main_query()` and `!is_admin()`.

- **Q: How to paginate a custom `WP_Query`?**
  A: Pass `paged` => `get_query_var('paged')` and then use `$q->max_num_pages` with `paginate_links()` (or `the_posts_pagination()` with set globals).

- **Q: How to optimize a heavy query?**
  A: Use `no_found_rows`, `fields => 'ids'`, proper caching (transients/object cache), prefer taxonomy over meta for indexing; consider custom DB table if at scale.

- **Q: Best way to implement Game ↔ Casino relationships?**
  A: For simple many-to-many, use ACF relationship or a taxonomy if casino can be term. For scale or structured casino data, use a custom relationship table or a relationship plugin (Meta Box Relationships).

---

### Managing relationships between CPTs (Game ↔ Casino)

Common approaches (trade-offs):

1. **Taxonomy** (fast, built-in): If a casino can be treated as a term and you don’t need lots of structured fields, register a taxonomy (`casino`) and attach to `game`.

   - Pros: fast, built-in UI, good for many-to-many-like queries.
   - Cons: limited structure for casino fields.

2. **Post meta (storing arrays / ACF relationship)**: Use ACF Relationship or a meta field `related_games` on casino.

   - Pros: easy UI (ACF), straightforward retrieval.
   - Cons: stored in postmeta (less performant at scale), harder to query natively.

3. **Custom join table (best at scale)**: Create a DB table `wp_game_casino` with `game_id, casino_id`. Query with `$wpdb` and then run `WP_Query` with `post__in`.

   - Pros: scalable, indexed queries.
   - Cons: more complex (migration, backups), require custom UI.

4. **Plugins**: MB Relationships (Meta Box), or other relationship frameworks — they offer admin UI + optimized queries.

Example retrieval with ACF relationship (recommended for developer productivity):

```php
$games = get_field('related_games', $casino_id); // returns array of post objects or IDs
if ($games) {
  foreach ($games as $game) {
    // $game is either ID or WP_Post depending on ACF settings
  }
}
```

---

### Core concepts & functions (the essentials)

- **❗️Main Query vs Custom Query**

  - **Main query**: the query WP runs to decide what page you’re on (is_archive, is_single, etc.). Accessible via global `$wp_query` or `is_main_query()` inside hooks.
  - **Custom query**: created with `new WP_Query()` or `get_posts()`. Use for secondary loops.

- **❗️Primary Loop functions**

  - `have_posts()` — returns true if there’s another post in the current query.
  - `the_post()` — advances internal pointer and sets up post global data (makes template tags work).
  - Template tags: `the_title()`, `the_content()`, `the_excerpt()`, `the_permalink()`, etc.

- **❗️Reset functions**

  - `wp_reset_postdata()` — use after a `WP_Query` loop to restore global `$post`.
  - `wp_reset_query()` — resets `$wp_query` and global query vars (used when you used `query_posts()`, which you should avoid).

- **❗️Query tools**

  - `WP_Query` — most flexible, returns object with `have_posts()`, `max_num_pages`, etc.
  - `get_posts()` — wrapper returning array of posts; lighter but less flexible (suitable for small sets).
  - `query_posts()` — modifies main query but _replaces_ it (bad practice). Avoid.

---

### Common mistakes to avoid (say these in interview to show depth)

- Using `query_posts()` in templates (causes subtle bugs).
- Forgetting `wp_reset_postdata()` after custom queries.
- Overusing meta queries on large datasets without caching or DB design.
- Not sanitizing shortcode attributes or escaping output.
- Using `posts_per_page => -1` for huge tables (OOM risk); paginate instead.

---

# Loops

### 1. Default Loop - (`have_posts()`)

**Purpose:** The main loop that displays posts on blog index, archives, categories, or any default WordPress query.
**Common Use Cases:**

- Showing blog posts on homepage or archive pages
- Listing posts under a category or tag page

**Best Practices:**

- Always use `have_posts()` and `the_post()` for iteration
- Include an `else` fallback for “no posts” situations
- Wrap output in semantic HTML like `<article>` or `<section>`

**Code Examples:**

```php
// 1. Titles only
if ( have_posts() ) :
  while ( have_posts() ) : the_post();
    the_title('<h2>', '</h2>');
  endwhile;
else:
  echo 'No posts found.';
endif;
```

```php
// 2. Title + excerpt
if ( have_posts() ) :
  while ( have_posts() ) : the_post();
    the_title('<h2>', '</h2>');
    the_excerpt();
  endwhile;
endif;
```

```php
// 3. Title with permalink
<php?
if ( have_posts() ) :
  while ( have_posts() ) : the_post(); ?>
    <h2><a href="<?php the_permalink(); ?>"><?php the_title(); ?></a></h2>
  <?php endwhile;
endif;
?>
```

[Back to Top ⬆️](#-wordpress-loops)

### 2. Custom Query Loop (`WP_Query`)

**Purpose:** Runs custom database queries without interfering with the main loop, allowing for advanced filtering.
**Common Use Cases:**

- Fetching featured posts or recent posts for widgets
- Displaying custom post types or filtered posts

**Best Practices:**

- Always reset post data with `wp_reset_postdata()`
- Use clear and specific query arguments to avoid performance issues
- Avoid complex queries inside template files; consider caching results

**Code Examples:**

```php
// 1. Latest 5 posts
$query = new WP_Query(['posts_per_page' => 5]);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
wp_reset_postdata();

// 2. Posts from category ID 10
$query = new WP_Query(['cat' => 10]);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
wp_reset_postdata();

// 3. Custom post type portfolio
$query = new WP_Query(['post_type' => 'portfolio', 'posts_per_page' => 3]);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
wp_reset_postdata();
```

[Back to Top ⬆️](#-wordpress-loops)

### 3. Secondary Loop (Nested Loops)

**Purpose:** Runs multiple queries on the same page without disturbing the main content.
**Common Use Cases:**

- Showing related posts after the main post
- Sidebars with recent posts or comments

**Best Practices:**

- Reset post data after each secondary loop using `wp_reset_postdata()`
- Avoid nesting loops too deeply to keep performance healthy

**Code Examples:**

```php
// 1. Related posts by category
if ( have_posts() ) : while ( have_posts() ) : the_post();
  the_title('<h1>', '</h1>');

  $related = new WP_Query([
    'category__in' => wp_get_post_categories(get_the_ID()),
    'posts_per_page' => 3,
    'post__not_in' => [get_the_ID()]
  ]);
  while ( $related->have_posts() ) : $related->the_post();
    the_title('<h4>', '</h4>');
  endwhile;
  wp_reset_postdata();
endwhile; endif;

// 2. Main loop + sidebar posts
if ( have_posts() ) : while ( have_posts() ) : the_post();
  the_title();
endwhile; endif;

$sidebar = new WP_Query(['posts_per_page' => 2]);
while ( $sidebar->have_posts() ) : $sidebar->the_post();
  the_title('<p>', '</p>');
endwhile;
wp_reset_postdata();

// 3. Main post + recent comments
if ( have_posts() ) : while ( have_posts() ) : the_post();
  the_title();
endwhile; endif;

$comments = get_comments(['number' => 3]);
foreach ( $comments as $comment ) {
  echo '<p>' . esc_html($comment->comment_author) . ': ' . esc_html($comment->comment_content) . '</p>';
}
```

[Back to Top ⬆️](#-wordpress-loops)

### 4. `query_posts()` Loop

**Purpose:** Modifies the main query but generally discouraged in favor of `WP_Query` or hooks.
**Common Use Cases:**

- Legacy themes or quick query override

**Best Practices:**

- Avoid if possible; prefer `pre_get_posts` or custom `WP_Query`
- Always reset query with `wp_reset_query()` after use

**Code Examples:**

```php
// 1. Show 3 posts only
query_posts(['posts_per_page' => 3]);
while ( have_posts() ) : the_post();
  the_title();
endwhile;
wp_reset_query();

// 2. Posts from category 5
query_posts(['cat' => 5]);
while ( have_posts() ) : the_post();
  the_title();
endwhile;
wp_reset_query();

// 3. Posts by author 2
query_posts(['author' => 2]);
while ( have_posts() ) : the_post();
  the_title();
endwhile;
wp_reset_query();
```

[Back to Top ⬆️](#-wordpress-loops)

### 5. `get_posts()` Loop

**Purpose:** Quickly fetch a simple array of posts; less overhead than `WP_Query`.
**Common Use Cases:**

- Small post lists in widgets or custom blocks

**Best Practices:**

- Call `setup_postdata()` before template tags
- Always reset with `wp_reset_postdata()` afterward

**Code Examples:**

```php
// 1. Latest 5 posts
$posts = get_posts(['numberposts' => 5]);
foreach ( $posts as $post ) {
  setup_postdata( $post );
  the_title();
}
wp_reset_postdata();

// 2. Category 3 posts
$posts = get_posts(['category' => 3]);
foreach ( $posts as $post ) {
  setup_postdata( $post );
  the_title();
}
wp_reset_postdata();

// 3. Portfolio CPT posts
$posts = get_posts(['post_type' => 'portfolio', 'numberposts' => 3]);
foreach ( $posts as $post ) {
  setup_postdata( $post );
  the_title();
}
wp_reset_postdata();
```

[Back to Top ⬆️](#-wordpress-loops)

### 6. Taxonomy Loop

**Purpose:** Retrieve and list taxonomy terms like categories or tags.
**Common Use Cases:**

- Display category lists, tag clouds, or custom taxonomy filters

**Best Practices:**

- Use `hide_empty` argument wisely
- Cache results for performance

**Code Examples:**

```php
// 1. List all categories
$terms = get_terms(['taxonomy' => 'category', 'hide_empty' => false]);
foreach ( $terms as $term ) {
  echo esc_html($term->name);
}

// 2. List all tags
$tags = get_terms(['taxonomy' => 'post_tag']);
foreach ( $tags as $tag ) {
  echo esc_html($tag->name);
}

// 3. List custom taxonomy 'genre'
$genres = get_terms(['taxonomy' => 'genre', 'hide_empty' => true]);
foreach ( $genres as $genre ) {
  echo esc_html($genre->name);
}
```

[Back to Top ⬆️](#-wordpress-loops)

### 7. Custom Post Type (CPT) Loop

**Purpose:** Query and display posts from custom post types.
**Common Use Cases:**

- Display portfolios, events, products, or other CPT content

**Best Practices:**

- Register CPTs with proper supports and archive flags
- Use explicit `post_type` in queries

**Code Examples:**

```php
// 1. List all portfolio items
$query = new WP_Query(['post_type' => 'portfolio']);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
wp_reset_postdata();

// 2. Latest 3 events
$query = new WP_Query(['post_type' => 'event', 'posts_per_page' => 3]);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
wp_reset_postdata();

// 3. Paginated products
$paged = get_query_var('paged') ?: 1;
$query = new WP_Query([
  'post_type' => 'product',
  'posts_per_page' => 5,
  'paged' => $paged
]);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
wp_reset_postdata();
```

[Back to Top ⬆️](#-wordpress-loops)

### 8. Meta Query Loop

**Purpose:** Filter posts by custom fields/meta data.
**Common Use Cases:**

- Filter products by price, events by date, posts marked as featured

**Best Practices:**

- Use correct meta types and comparison operators
- Index meta keys on large sites for speed

**Code Examples:**

```php
// 1. Featured posts
$query = new WP_Query([
  'meta_query' => [
    ['key' => 'featured', 'value' => 'yes', 'compare' => '=']
  ]
]);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
wp_reset_postdata();

// 2. Products priced above 100
$query = new WP_Query([
  'meta_query' => [
    ['key' => 'price', 'value' => 100, 'compare' => '>', 'type' => 'NUMERIC']
  ]
]);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
wp_reset_postdata();

// 3. Events after today
$query = new WP_Query([
  'meta_query' => [
    ['key' => 'event_date', 'value' => date('Y-m-d'), 'compare' => '>=', 'type' => 'DATE']
  ]
]);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
wp_reset_postdata();
```

[Back to Top ⬆️](#-wordpress-loops)

### 9. Date Query Loop

**Purpose:** Filter posts by date ranges.
**Common Use Cases:**

- Archive pages by month, year or custom date ranges
- Upcoming events

**Best Practices:**

- Use `inclusive` to include boundary dates
- Combine with pagination

**Code Examples:**

```php
// 1. Posts from 2024
$query = new WP_Query([
  'date_query' => [['year' => 2024]]
]);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
wp_reset_postdata();

// 2. Posts between Jan-Mar 2024
$query = new WP_Query([
  'date_query' => [['after' => 'January 1 2024', 'before' => 'March 31 2024', 'inclusive' => true]]
]);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
wp_reset_postdata();

// 3. Posts from last 7 days
$query = new WP_Query([
  'date_query' => [['after' => '7 days ago']]
]);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
wp_reset_postdata();
```

[Back to Top ⬆️](#-wordpress-loops)

### 10. Author Loop

**Purpose:** Show posts by specific author(s).
**Common Use Cases:**

- Author archive pages
- Team member portfolios

**Best Practices:**

- Sanitize author IDs
- Combine with pagination

**Code Examples:**

```php
// 1. Posts by author ID 1
$query = new WP_Query(['author' => 1]);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
wp_reset_postdata();

// 2. Posts by authors 1 and 2
$query = new WP_Query(['author__in' => [1, 2]]);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
wp_reset_postdata();

// 3. Posts excluding author 3
$query = new WP_Query(['author__not_in' => [3]]);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
wp_reset_postdata();
```

[Back to Top ⬆️](#-wordpress-loops)

### 11. Search Results Loop

**Purpose:** Show results matching user search queries.
**Common Use Cases:**

- Search results page
- Custom search widgets

**Best Practices:**

- Sanitize search input
- Handle no-results gracefully

**Code Examples:**

```php
// 1. Search for "WordPress"
$query = new WP_Query(['s' => 'WordPress']);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
wp_reset_postdata();

// 2. Search in CPT 'product' for 'laptop'
$query = new WP_Query(['post_type' => 'product', 's' => 'laptop']);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
wp_reset_postdata();

// 3. Search in category 4 for 'SEO'
$query = new WP_Query(['cat' => 4, 's' => 'SEO']);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
wp_reset_postdata();
```

[Back to Top ⬆️](#-wordpress-loops)

### 12. Pagination Loop

**Purpose:** Divide post listings into pages for better UX.
**Common Use Cases:**

- Blog archives
- CPT archives

**Best Practices:**

- Use `get_query_var('paged')`
- Use pagination functions (`paginate_links`, `previous_posts_link`)
- Avoid loading all posts at once

**Code Examples:**

```php
// 1. Basic blog pagination
$paged = get_query_var('paged') ?: 1;
$query = new WP_Query(['posts_per_page' => 5, 'paged' => $paged]);
while ( $query->have_posts() ) : $query->the_post();
  the_title();
endwhile;
previous_posts_link('« Newer');
next_posts_link('Older »', $query->max_num_pages);
wp_reset_postdata();

// 2. Pagination for portfolio CPT
$paged = get_query_var('paged') ?: 1;
$query = new WP_Query(['post_type']);
```

[Back to Top ⬆️](#-wordpress-loops)
