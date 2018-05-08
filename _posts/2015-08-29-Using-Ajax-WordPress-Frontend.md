---
layout: post
title: Using AJAX on the WordPress Frontend
---

There are a few helpful things to bear in mind when using ajax on the frontend:

Actions must be hooked up before the ‘wp’ hook (I usually run them on ‘init’)
The ajaxurl javascript global may not be defined and so you need to do this yourself
There are two hooks to use for the ajax call
This code example is a good place to start (tweaked slightly from the codex). Firstly, we enqueue our JS file making sure we define any variables needed (ajaxurl is essential). Then we hook up our ajax method twice (these actions run before the ‘wp’ hook remember) and die to finish:

``add_action( 'admin_enqueue_scripts', 'my_enqueue' );``
``
function my_enqueue( $hook ) {
  wp_enqueue_script( 'ajax-script', plugins_url( '/js/my_query.js', \__FILE\__ ), array( 'jquery' ) );
  wp_localize_script( 'ajax-script', 'ajax_object',
    array( 'ajax_url' => admin_url( 'admin-ajax.php' ), 'we_value' => 1234 )
  );
}
``

``
add_action( 'wp_ajax_my_action', 'my_action_callback' );
add_action( 'wp_ajax_nopriv_my_action', 'my_action_callback' );
``
``
function my_action_callback() {
  global $wpdb;
  $whatever = intval( $_POST['whatever'] );
  $whatever += 10;
  echo $whatever;
  wp_die();
}``

And our JS file simply grabs the values we passed and calls our action passing in any data we require:

``
jQuery( document ).ready( function( $ ) {
  var data = {
    'action': 'my;_action',
		'whatever': ajax_object.we_value
	};
	jQuery.post( ajax_object.ajax_url, data, function( response ) {
		alert( 'Got this from the server: ' + response );
	});
});
``

If your alert box shows only a 0 then your action is not being called. Check your hooks are firing and all function names are correct.
