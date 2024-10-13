## 📌 Custom Queries

Custom Queries allow us to load whatever we want, wherever we want.

#### Understanding Custom Queries in WordPress

In WordPress, a Custom Query refers to a query that retrieves specific data from the WordPress database beyond the default set of posts or pages. It allows developers to fetch data based on custom criteria such as custom post types, specific taxonomies, meta fields, or a particular date range. Custom queries give you the flexibility to control what content is displayed and how it is organized, making them an essential feature for any WordPress developer looking to create a more dynamic and customized user experience.

#### What Is a Custom Query?

A **custom query** in WordPress refers to a query that fetches specific data based on criteria that you define, rather than relying on the default behavior of WordPress. This means you can display data such as custom post types, specific categories, tags, meta fields, or even posts from a specific time range. Custom queries allow you to do things that are not directly tied to the current page or URL.

So, custom queries are the right tool for the job when you want to do something that isn't related to the default behavior of the current URL.


The **WP_Query** class in WordPress is the main class used to execute custom queries, but if you need more control over what content is displayed, you can also use other functions like **get_posts()** and **query_posts()**.

`WP_Query()` - Sets up the WordPress query, if parameter is not empty.

### Core Concepts of Custom Queries

1. **WP_Query**
   - `WP_Query` is a class that you can use to create your custom queries. It offers a wide range of parameters to help you fetch exactly the data you need.
   - You can use arguments like:
     - **post_type**: Filter posts by custom post types (e.g., 'post', 'page', 'product', 'event').
     - **posts_per_page**: Limit the number of posts returned.
     - **orderby**: Sort posts by date, title, custom fields, and more.
     - **meta_query**: Perform complex queries based on custom fields.

2. **get_posts()**
   - The `get_posts()` function provides a simplified way to retrieve posts, using fewer parameters than WP_Query.
   - It's generally used when you want to fetch a straightforward list of posts without setting up a full loop.

3. **query_posts()**
   - `query_posts()` modifies the main query of a page, which can have side effects on the global $wp_query object.
   - This approach is less recommended because it can conflict with WordPress's main query and lead to unexpected results.

### Purpose of Custom Queries

The main purpose of custom queries is to give you greater control over the data displayed on your WordPress site. Here are some scenarios where you might use a custom query:

- **Displaying Custom Post Types**: If you have custom post types like 'products,' 'events,' or 'portfolio,' you can create a custom query to display these items on a specific page.
- **Filtering Content**: You can use custom queries to filter content by taxonomy, category, tag, or even custom fields (meta queries).
- **Creating Dynamic Widgets**: Widgets like recent posts, featured posts, or most popular posts can be powered by custom queries.
- **Building Complex Page Layouts**: Custom queries allow you to create unique page layouts where specific content appears based on various conditions.

### Use Cases of Custom Queries in Practice

1. **Displaying Events Based on Custom Post Type**
   Suppose you have a custom post type called 'events' and want to display them in a chronological order on the homepage:

   ```php
   <?php 
       $homepageEvents = new WP_Query(array(
           'posts_per_page' => -1,
           'post_type' => 'events',
           'orderby' => 'post_date',
           'order' => 'ASC'
       )); 

   while($homepageEvents->have_posts()){
       $homepageEvents->the_post();
   ?>
       <div class="event-summary">
       <a class="event-summary__date event-summary__date--beige t-center" href="<?php the_permalink(); ?>">
           <span class="event-summary__month"><?php the_time('M'); ?></span>
           <span class="event-summary__day"><?php the_time('d'); ?></span>
       </a>
       <div class="event-summary__content">
           <h5 class="event-summary__title headline headline--tiny"><a href="<?php the_permalink(); ?>"><?php the_title(); ?></a></h5>
           <p><?php echo wp_trim_words(get_the_content(), 18); ?> <a href="<?php the_permalink(); ?>" class="nu gray">Read more</a></p>
       </div>
       </div>
   <?php } wp_reset_postdata(); ?> 
   ```

   In this example:
   - **posts_per_page**: `-1` ensures all events are listed.
   - **post_type**: Fetches only posts from the custom post type 'events'.
   - **orderby** and **order**: Sorts the events chronologically in ascending order.

2. **Fetching Posts from a Specific Category**
   To display posts only from the 'News' category, you could use the following code:

   ```php
   <?php
   $newsPosts = new WP_Query(array(
       'category_name' => 'news',
       'posts_per_page' => 5
   ));

   while ($newsPosts->have_posts()) {
       $newsPosts->the_post();
       echo '<h2>' . get_the_title() . '</h2>';
   }
   wp_reset_postdata();
   ?>
   ```

3. **Using Meta Queries to Filter Posts**
   Suppose you want to display products priced above a certain amount using meta queries:

   ```php
   <?php
   $expensiveProducts = new WP_Query(array(
       'post_type' => 'product',
       'meta_query' => array(
           array(
               'key' => 'price',
               'value' => 100,
               'compare' => '>',
               'type' => 'NUMERIC'
           )
       )
   ));

   while ($expensiveProducts->have_posts()) {
       $expensiveProducts->the_post();
       echo '<h2>' . get_the_title() . '</h2>';
   }
   wp_reset_postdata();
   ?>
   ```

### Best Practices for Using Custom Queries

1. **Use WP_Query for Complex Queries**: When you need a more advanced query with various parameters, always use `WP_Query` to handle your requests.
2. **Avoid Using query_posts()**: Since `query_posts()` modifies the global query, it’s best to avoid it. Use `WP_Query` instead to prevent conflicts.
3. **Reset Post Data**: Always reset the post data using `wp_reset_postdata()` after running a custom loop to ensure that subsequent queries or template tags work correctly.
4. **Caching Queries**: If you're running complex queries on high-traffic pages, consider caching the results to improve performance.

### Benefits of Using Custom Queries

- **Flexibility**: Custom queries allow for endless possibilities in content retrieval.
- **Enhanced User Experience**: They let you present content in a way that suits your audience’s needs.
- **Performance Optimization**: By displaying only the data you need, you can reduce page load times and enhance website performance.

Custom queries are a fundamental aspect of WordPress development that gives you control over how content is displayed on your website. Understanding how to use and optimize them is key to creating powerful and engaging WordPress sites.