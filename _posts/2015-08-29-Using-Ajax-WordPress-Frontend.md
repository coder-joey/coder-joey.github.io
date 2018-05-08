---
layout: post
title: Using AJAX on the WordPress Frontend
---

There are a few helpful things to bear in mind when using ajax on the frontend:

Actions must be hooked up before the ‘wp’ hook (I usually run them on ‘init’)
The ajaxurl javascript global may not be defined and so you need to do this yourself
There are two hooks to use for the ajax call
This code example is a good place to start (tweaked slightly from the codex). Firstly, we enqueue our JS file making sure we define any variables needed (ajaxurl is essential). Then we hook up our ajax method twice (these actions run before the ‘wp’ hook remember) and die to finish:

<code>add_action( 'admin_enqueue_scripts', 'my_enqueue' );</code><br />
<code>function my_enqueue( $hook ) {<br />
  wp_enqueue_script( 'ajax-script', plugins_url( '/js/my_query.js', \__FILE\__ ), array( 'jquery' ) );<br />
  wp_localize_script( 'ajax-script', 'ajax_object',<br />
    array( 'ajax_url' => admin_url( 'admin-ajax.php' ), 'we_value' => 1234 )<br />
  );<br />
}</code>
<code>add_action( 'wp_ajax_my_action', 'my_action_callback' );<br />
add_action( 'wp_ajax_nopriv_my_action', 'my_action_callback' );</code><br />
<code>function my_action_callback() {<br />
  global $wpdb;<br />
  $whatever = intval( $_POST['whatever'] );<br />
  $whatever += 10;<br />
  echo $whatever;<br />
  wp_die();<br />
}</code>

And our JS file simply grabs the values we passed and calls our action passing in any data we require:

<code>jQuery( document ).ready( function( $ ) {<br />
  var data = {<br />
    'action': 'my;_action',<br />
		'whatever': ajax_object.we_value<br />
	};<br />
	jQuery.post( ajax_object.ajax_url, data, function( response ) {<br />
		alert( 'Got this from the server: ' + response );<br />
	});<br />
});</code>

If your alert box shows only a 0 then your action is not being called. Check your hooks are firing and all function names are correct.
