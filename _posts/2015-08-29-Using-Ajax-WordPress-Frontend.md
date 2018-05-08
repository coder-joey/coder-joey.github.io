---
layout: post
title: Using AJAX on the WordPress Frontend
---

There are a few helpful things to bear in mind when using ajax on the frontend:

Actions must be hooked up before the ‘wp’ hook (I usually run them on ‘init’)
The ajaxurl javascript global may not be defined and so you need to do this yourself
There are two hooks to use for the ajax call
This code example is a good place to start (tweaked slightly from the codex). Firstly, we enqueue our JS file making sure we define any variables needed (ajaxurl is essential). Then we hook up our ajax method twice (these actions run before the ‘wp’ hook remember) and die to finish:

<code>
add_action( 'admin_enqueue_scripts', 'my_enqueue' );\n
function my_enqueue( $hook ) {\n
  wp_enqueue_script( 'ajax-script', plugins_url( '/js/my_query.js', \__FILE\__ ), array( 'jquery' ) );\n
  wp_localize_script( 'ajax-script', 'ajax_object',\n
    array( 'ajax_url' => admin_url( 'admin-ajax.php' ), 'we_value' => 1234 )\n
  );\n
}
</code>
<code>
add_action( 'wp_ajax_my_action', 'my_action_callback' );\n
add_action( 'wp_ajax_nopriv_my_action', 'my_action_callback' );\n
function my_action_callback() {\n
  global $wpdb;\n
  $whatever = intval( $_POST['whatever'] );\n
  $whatever += 10;\n
  echo $whatever;\n
  wp_die();\n
}
</code>

And our JS file simply grabs the values we passed and calls our action passing in any data we require:

<code>
jQuery( document ).ready( function( $ ) {
  var data = {
    'action': 'my;\_action',
		'whatever': ajax\_object.we\_value
	};
	jQuery.post( ajax\_object.ajax\_url, data, function( response ) {
		alert( 'Got this from the server: ' + response );
	});
});
</code>

If your alert box shows only a 0 then your action is not being called. Check your hooks are firing and all function names are correct.
