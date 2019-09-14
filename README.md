# Enhancing your WP Admin Panel

## Slides

- [enhancing-wp-admin-panel-wczurich2019.pdf](https://github.com/alexgordon25/wczrh2019/raw/master/enhancing-wp-admin-panel-wczurich2019.pdf)

## Add Styles to your login screen

```
<?php
//Custom CSS for login page
function login_css() {
    wp_enqueue_style('login-page',
         get_template_directory_uri() . '/css/login.css', false );
}

// Change logo link from wordpress.org to your site’s url
function login_url() { return home_url(); }

// Change the alt text on the logo to show your site’s name
function login_title() { return get_option('blogname'); }

// calling it only on the login page
add_action( 'login_enqueue_scripts', 'login_css', 10 );
add_filter( 'login_headerurl', 'login_url');
add_filter( 'login_headertitle', 'login_title');
?>
```

## Promote your Dev Services

```
<?php
// Custom Backend Footer
function custom_admin_footer() {
	?>
	<span id="footer-thankyou">Development by <a href="https://twitter.com/alexgordon25" target="_blank">Daniel Gordon</a>. For any web development assitance on this site <a href="mailto:daniel@gordonwebstudio.com">Contact me</a>.<span>
	<?php
}

// adding it to the admin area
add_filter('admin_footer_text', 'custom_admin_footer');
?>
```

## Expand the info in your listing

### Add columns

```
<?php
/* Adds new column headers to 'movies' post type */
function movies_add_new_columns($columns)  {
        //Remove irrelevant columns
        unset($columns['author']);
        unset($columns['comments']);
        unset($columns['date']);

	$columns['release'] = 'Release Date';
	return $columns;
}
add_filter('manage_edit-movies_columns', 'movies_add_new_columns');

/* Adds content to new columns for 'movies' post type */
function movies_manage_columns($column_name, $id) {
	global $post;
	switch ($column_name) {
		case 'release':
			echo get_post_meta( $post->ID , 'release-year' , true );
			break;
	}
}
add_action('manage_movies_posts_custom_column', 'movies_manage_columns', 10, 2);
?>
```

### Add a category filter selector

```
<?php
/* Adds a filter in the Admin list pages on taxonomy for easier locating */
function movie_genre_filter_admin_content() {
    global $typenow;
    if( $typenow == "movies") { $taxonomies = array('genre'); }
    if (isset($taxonomies)) {
        foreach ($taxonomies as $tax_slug) {
            $tax_obj = get_taxonomy($tax_slug);
            $tax_name = $tax_obj->labels->name;
            $terms = get_terms($tax_slug);
            echo "<select name='$tax_slug' id='$tax_slug' class='postform'>";
            echo "<option value=''>Show All $tax_name</option>";
            foreach ($terms as $term) {
                $label = (isset($_GET[$tax_slug])) ? $_GET[$tax_slug] : '';
                echo '<option value='. $term->slug,
                        $label == $term->slug ? ' selected="selected"' : '','>' .
                        $term->name .'</option>';
            }
        }
    }
}
add_action( 'restrict_manage_posts', 'movie_genre_filter_admin_content' );
?>
```

## Add styles to the Content Editor

```
<?php
// Apply styles to content editor to make it match the site
function add_custom_editor_style() {
    //Looks for an 'editor-style.css' in your theme folder
    add_editor_style();

    //Or call it with a custom file name if you choose
    //(helpful especially when keeping css in a subfolder)
    //add_editor_style('css/my-editor-style.css');
}
add_action( 'admin_init', 'add_custom_editor_style' );
?>
```

For the block editor in Gutenberg add `editor-styles` theme support.

```
add_theme_support('editor-styles');
```

## Add Google Fonts to Content Editor.

```
<?php
/* Add Google Fonts to the Admin editor */
function add_google_fonts_admin_editor() {
   $font_url = 'https://fonts.googleapis.com/css?family=Oswald:400,700';

   //Encode Google Fonts URL if it contains commas
   add_editor_style( esc_url( $font_url ) );
}
add_action( 'admin_init', 'add_google_fonts_admin_editor' );
?>
```

## Styles Selector for TinyMCE

```
<?php
/*
 * Adding new TinyMCE styles to editor
 * https://codex.wordpress.org/TinyMCE_Custom_Styles
 */

// Callback function to insert 'styleselect' into the $buttons array
function add_tiny_mce_buttons( $buttons ) {
	array_unshift( $buttons, 'styleselect' );
	return $buttons;
}
// Register our callback to the appropriate filter (2 means 2nd row)
add_filter('mce_buttons_2', 'add_tiny_mce_buttons');

// Callback function to filter the Tiny MCE settings
function tiny_mce_add_styles( $init_array ) {
	// Define the style_formats array
	$style_formats = array(
		// Each array child is a format with its own settings
		array(
			'title' => 'Blue Headline',
			'block' => 'div',
			'classes' => 'blue-headline'
		)
 	);
	// Insert the array, JSON ENCODED, into 'style_formats'
	$init_array['style_formats'] = json_encode( $style_formats );

	return $init_array;
}
add_filter('tiny_mce_before_init','tiny_mce_add_styles' );
?>
```

## Add new Widget to WP Dashboard

```
<?php
//Add widget to dashboard page
function add_movies_dashboard_widgets() {
   wp_add_dashboard_widget('movies-widget', 'Recent Movies', 'movies_dashboard_widget');
}
add_action('wp_dashboard_setup', 'add_movies_dashboard_widgets');

//Dashboard widget logic (can be anything!)
function movies_dashboard_widget() {
    $args = array ( 'post_type' => 'movies', 'posts_per_page' => 5 );
    $movies = new WP_Query( $args );

    if($movies->have_posts()) :
        while($movies->have_posts()): $movies->the_post();
            echo '<div style="margin-bottom: 10px;">';
            $url = home_url() . "/wp-admin/post.php?post=" . get_the_id() . '&action=edit';
            echo '<a href="' . $url . '">' . get_the_title() . '</a> - ';
            echo get_post_meta( get_the_id(), 'release-year' , true ) . '</div>';
        endwhile;
    endif;
}
?>
```

## Add a link to the Admin Bar

```
<?php
//Add Links to Admin Bar
function movie_admin_bar_link( $wp_admin_bar ) {
    //Only add link for Admins
    if (current_user_can( 'manage_options' )) {
	$args = array(
	    'id'    => 'movies-page',
	    'title' => 'Manage Movies',
	    'href'  => home_url() . '/wp-admin/edit.php?post_type=movies',
	    'meta'  => array( 'class' => 'movies-admin-link' )
	);
        $wp_admin_bar->add_node( $args );
    }
}
add_action( 'admin_bar_menu', 'movie_admin_bar_link', 100 );
?>
```
