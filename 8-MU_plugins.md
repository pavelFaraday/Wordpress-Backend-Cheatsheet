## 📌 Must Use Plugins (MU-Plugins) in WordPress

In WordPress, "Must-Use Plugins" (also known as MU-Plugins) are special types of plugins that are automatically enabled for a site without needing to be manually activated. These plugins are different from regular plugins in the following ways:

Key Features of Must-Use Plugins:
1. **Automatic Activation:** Must-use plugins are automatically activated when placed in the mu-plugins directory. You cannot deactivate them through the WordPress admin dashboard.

2. **Special Directory:** MU-plugins are stored in a specific folder located at `wp-content/mu-plugins/`. This is separate from the standard plugin folder.

3. **Global Across Multisites:** For WordPress multisite installations, MU-plugins can be used across all sites in the network. This is especially useful for plugins that need to be applied universally, like security or performance-related plugins.

4. **No UI in Admin:** Must-use plugins don't appear in the standard "Plugins" section of the WordPress admin dashboard. Instead, they are listed under a separate "Must-Use" tab, if present.

5. **Use Cases:** They are typically used for essential functionality that should always be active, such as security features, caching systems, or specific custom code that needs to run regardless of other plugin activations.

6. **No Deactivation:** Since these plugins are managed manually by adding or removing files from the mu-plugins folder, they can't be turned off through the dashboard.

#### How to Add a Must-Use Plugin:
To add a must-use plugin, simply upload the PHP file (or a directory containing the plugin's files) into the wp-content/mu-plugins/ folder. The plugin will automatically be enabled.

This is commonly used by developers to ensure critical functionality is always enabled and cannot be accidentally disabled by admins.

---