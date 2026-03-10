## Review: Missing permission_callback in REST API Route

When using register_rest_route() or wp_register_ability() to define custom REST API endpoints, it is crucial to include a proper permission_callback .

🔒 This callback function ensures that only authorized users can access or modify data through your endpoint.

Code example, checking that the user can change options:
register_rest_route( 'wishcart-wishlist-for-fluentcart/v1', '/my-endpoint', array(
    'methods' => 'GET',
    'callback' => 'wishcart-wishlist-for-fluentcart_callback_function',
    'permission_callback' => function() {
        return current_user_can( 'manage_options' );
    }
) );

Please check the register_rest_route() documentation and the current_user_can() documentation.

✅ When a permission_callback is NOT Required:

There are valid use cases for public endpoints, such as publicly available data (e.g., posts, public metadata) or endpoints designed for unauthenticated access (e.g., fetching public stats or information).

In these cases, you should use __return_true as the permission_callback to indicate that the endpoint is intentionally public.

🔒 When a permission_callback IS Required:

For endpoints that involve sensitive data or actions (e.g., getting not public data, creating, updating, or deleting content).

In these cases, you should always implement proper permission checks.

Possible cases found on this plugin's code:
includes/class-wishcart-admin.php:606 register_rest_route('wishcart/v1', '/activity/wishlist/(?P<wishlist_id>\\d+)', array('methods' => 'GET', 'callback' => array($this, 'activity_get_wishlist'), 'permission_callback' => '__return_true', 'args' => array('wishlist_id' => array('description' => __('Wishlist ID', 'wishcart'), 'type' => 'integer', 'required' => true, 'sanitize_callback' => 'absint'), 'limit' => array('description' => __('Number of activities to return', 'wishcart'), 'type' => 'integer', 'required' => false, 'sanitize_callback' => 'absint', 'validate_callback' => function ($param) {
    return is_numeric($param) && $param > 0 && $param <= 100;
}, 'default' => 50), 'offset' => array('description' => __('Number of activities to skip', 'wishcart'), 'type' => 'integer', 'required' => false, 'sanitize_callback' => 'absint', 'validate_callback' => function ($param) {
    return is_numeric($param) && $param >= 0;
}, 'default' => 0))));
includes/class-wishcart-admin.php:364 register_rest_route('wishcart/v1', '/wishlist/remove', array('methods' => 'POST', 'callback' => array($this, 'wishlist_remove'), 'permission_callback' => '__return_true'));
includes/class-wishcart-admin.php:427 register_rest_route('wishcart/v1', '/guest/update-email', array('methods' => 'POST', 'callback' => array($this, 'guest_update_email'), 'permission_callback' => '__return_true'));
includes/class-wishcart-admin.php:382 register_rest_route('wishcart/v1', '/wishlist/check/(?P<product_id>\\d+)', array(
    'methods' => 'GET',
    'callback' => array($this, 'wishlist_check'),
    'permission_callback' => '__return_true',
    // Public endpoint
    'args' => array('product_id' => array('required' => true, 'type' => 'integer')),
));
includes/class-wishcart-admin.php:414 register_rest_route('wishcart/v1', '/wishlist/users', array('methods' => 'GET', 'callback' => array($this, 'wishlist_get_users'), 'permission_callback' => '__return_true'));
includes/class-wishcart-admin.php:394 register_rest_route('wishcart/v1', '/product/(?P<product_id>\\d+)/variants', array(
    'methods' => 'GET',
    'callback' => array($this, 'get_product_variants'),
    'permission_callback' => '__return_true',
    // Public endpoint
    'args' => array('product_id' => array('required' => true, 'type' => 'integer')),
));
includes/class-wishcart-admin.php:421 register_rest_route('wishcart/v1', '/guest/check-email', array('methods' => 'GET', 'callback' => array($this, 'guest_check_email'), 'permission_callback' => '__return_true'));
includes/class-wishcart-admin.php:568 register_rest_route('wishcart/v1', '/notifications/subscribe', array('methods' => 'POST', 'callback' => array($this, 'notifications_subscribe'), 'permission_callback' => '__return_true'));
... out of a total of 11 incidences.


## The link to the ajax endpoint may not work in some configurations.

When you link to the Ajax endpoint, you cannot assume that it's always located at wp-admin/admin-ajax.php . There are different configurations in which that won't work.

This means you can't link it statically, you have to use a function to determine its location, for example: admin_url( 'admin-ajax.php' );

Obviously you would need to execute that call on PHP and then pass the information to your JS file, you can do that using the wp_localize_script() function which is also useful for other uses like passing the nonce. Let me share an example:
function wish82wi_scripts() {
wp_enqueue_script( 'wish82wi-script', WISH82WI_PLUGIN_URL . 'js/script.js', array(), WISH82WI_VERSION );

wp_localize_script( 'wish82wi-script', 'wish82wi-ajax', array(
  'ajax_url' => admin_url('admin-ajax.php'),
  'nonce'  => wp_create_nonce( 'wish82wi-ajax-nonce' ),
));
}
add_action( 'wp_enqueue_scripts', 'wish82wi_scripts' );

Once you have this, you can later refer to your Ajax endpoint in the JS file by using the variable wish82wi-ajax.ajax_url variable. Please refer to the Ajax documentation: https://developer.wordpress.org/plugins/javascript/ajax/

Example(s) from your plugin:
src/lib/fluentcartCart.js:121 const url = window.location.origin + '/wp-admin/admin-ajax.php?' + urlParams.toString();
build/wishlist-frontend.js:7809  ...BB().toString());const d=window.location.origin+"/wp-admin/admin-ajax.php?"+l.toString(),u=await fetch(d,{method:"GET",credentials:"same-origin",headers:{"X-Requested-With":"XMLHttpRequest"}});if(!u.o...  ...variation_id",t),fetch(`${window.location.origin}/wp-admin/admin-ajax.php`,{method:"POST",body:l,credentials:"same-origin"}).then(d=>{d.ok||d.status===200?n({success:!0}):c(s+1)}).catch(()=>{c(s+1)})}... 
src/lib/fluentcartCart.js:593 fetch(`${window.location.origin}/wp-admin/admin-ajax.php`, {





## The URL(s) declared in your plugin seems to be invalid or does not work.

From your plugin:

Author URI: https://gowishcart.com/ - gowishcart-wishlist-for-fluentcart.php - Could not resolve host: gowishcart.com
Plugin URI: https://gowishcart.com - gowishcart-wishlist-for-fluentcart.php - Could not resolve host: gowishcart.com



## Determine files and directories locations correctly

WordPress provides several functions for easily determining where a given file or directory lives.

We detected that the way your plugin references some files, directories and/or URLs may not work with all WordPress setups. This happens because there are hardcoded references or you are using the WordPress internal constants.

Let's improve it, please check out the following documentation:

https://developer.wordpress.org/plugins/plugin-basics/determining-plugin-and-content-directories/

It contains all the functions available to determine locations correctly.

Most common cases in plugins can be solved using the following functions:
For where your plugin is located: plugin_dir_path() , plugin_dir_url() , plugins_url()
For the uploads directory: wp_upload_dir() (Note: If you need to write files, please do so in a folder in the uploads directory, not in your plugin directories).

Example(s) from your plugin:
gowishcart-wishlist-for-fluentcart.php:158 $is_installed = file_exists( trailingslashit( WP_PLUGIN_DIR ) . $plugin_file );
includes/class-gowishcart-admin.php:710 if ( file_exists( trailingslashit( WP_PLUGIN_DIR ) . $path ) ) {



ℹ️ In order to determine your plugin location, you would need to use the __FILE__ variable for this to work properly.
Note that this variable depends on the location of the file making the call. As this can create confusion, a common practice is to save its value in a define() in the main file of your plugin so that you don't have to worry about this.

Example: Your main plugin file.
define( 'GOWIWIFO_PLUGIN_FILE', __FILE__ );
define( 'GOWIWIFO_PLUGIN_DIR', plugin_dir_path( __FILE__ ) );
define( 'GOWIWIFO_PLUGIN_URL', plugin_dir_url( __FILE__ ) );

Example: Any file of your plugin.
require_once GOWIWIFO_PLUGIN_DIR . 'admin/class-init.php';


function gowiwifo_scripts() {
 wp_enqueue_script( 'gowiwifo-script', GOWIWIFO_PLUGIN_URL . 'js/script.js', array(), GOWIWIFO_VERSION );
 // Or alternatively
 wp_enqueue_script( 'gowiwifo-script', plugins_url( 'js/script.js', GOWIWIFO_PLUGIN_FILE ), array(), GOWIWIFO_VERSION );
}
add_action( 'wp_enqueue_scripts', 'gowiwifo_scripts' );


Example(s) from your plugin:
gowishcart-wishlist-for-fluentcart.php:158 $is_installed = file_exists( trailingslashit( WP_PLUGIN_DIR ) . $plugin_file );
includes/class-gowishcart-admin.php:710 if ( file_exists( trailingslashit( WP_PLUGIN_DIR ) . $path ) ) {



## Variables and options must be escaped when echo'd

Much related to sanitizing everything, all variables that are echoed need to be escaped when they're echoed, so it can't hijack users or (worse) admin screens. There are many esc_*() functions you can use to make sure you don't show people the wrong data, as well as some that will allow you to echo HTML safely.

At this time, we ask you escape all $-variables, options, and any sort of generated data when it is being echoed. That means you should not be escaping when you build a variable, but when you output it at the end. We call this 'escaping late.'

Besides protecting yourself from a possible XSS vulnerability, escaping late makes sure that you're keeping the future you safe. While today your code may be only outputted hardcoded content, that may not be true in the future. By taking the time to properly escape when you echo, you prevent a mistake in the future from becoming a critical security issue.

This remains true of options you've saved to the database. Even if you've properly sanitized when you saved, the tools for sanitizing and escaping aren't interchangeable. Sanitizing makes sure it's safe for processing and storing in the database. Escaping makes it safe to output.

Also keep in mind that sometimes a function is echoing when it should really be returning content instead. This is a common mistake when it comes to returning JSON encoded content. Very rarely is that actually something you should be echoing at all. Echoing is because it needs to be on the screen, read by a human. Returning (which is what you would do with an API) can be json encoded, though remember to sanitize when you save to that json object!

There are a number of options to secure all types of content (html, email, etc). Yes, even HTML needs to be properly escaped.

https://developer.wordpress.org/apis/security/escaping/

Remember: You must use the most appropriate functions for the context. There is pretty much an option for everything you could echo. Even echoing HTML safely.

Example(s) from your plugin:
includes/class-wishlist-frontend.php:392 wp_add_inline_style( 'gowishcart-wishlist-frontend', $sanitized_css );
# ↳ Detected origin: wp_strip_all_tags($custom_css)
# ↳ Remember to ALWAYS escape as LATE as possible as with a PROPER function for the context.
includes/class-wishlist-frontend.php:385 wp_add_inline_style( 'gowishcart-wishlist-frontend', $generated_css );
# ↳ Detected origin: implode(" ", $css_parts)
# ↳ Remember to ALWAYS escape as LATE as possible as with a PROPER function for the context.


✔️ You can check this using Plugin Check.


## Unsafe SQL calls

When making database calls, it's highly important to protect your code from SQL injection vulnerabilities. You need to update your code to use wpdb calls and prepare() with your queries to protect them.

Please review the following:
https://developer.wordpress.org/reference/classes/wpdb/#protect-queries-against-sql-injection-attacks
https://codex.wordpress.org/Data_Validation#Database
https://make.wordpress.org/core/2012/12/12/php-warning-missing-argument-2-for-wpdb-prepare/
https://ottopress.com/2013/better-know-a-vulnerability-sql-injection/
Example(s) from your plugin:
includes/class-database.php:234 $index_name = $column . '_idx';
includes/class-database.php:237 $this->wpdb->query("ALTER TABLE {$table_name} ADD INDEX {$index_name} ({$column})");
# You cannot add variables like "$column" directly to the SQL query.
# Using wpdb::prepare($query, $args) you will need to include placeholders for each variable within the query and include the variables in the second parameter.
# The SQL query needs to be included in a wpdb::prepare($query, $args) function.includes/class-database.php:230 $this->wpdb->query($sql);
# The SQL query needs to be included in a wpdb::prepare($query, $args) function.
# Remember that you will need to include placeholders for each variable within the query and include their calls in the second parameter of wpdb::prepare().

