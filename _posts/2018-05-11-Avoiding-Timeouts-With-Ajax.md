---
layout: post
title: Avoiding Timeouts With Ajax
---

When developing plugins for WordPress I\'ve recently hit a couple of occasions where there was a need to update all posts of a certain type (in my case WooCommerce products).
On my local machine, even with a lot of products generated (5000+), I hadn\'t hit any timeout issues so it never occurred to me that this would become a problem.  But before too long I had customers attempting the same processes and running into issues (due to either server setup, size of database or other extension or similar).

I already had these processes running in ajax (triggered by a button located on the plugin\'s settings page) but I was running the events concurrently. For example, I was returning all products as one call, then running my heavier functions on 10 of these products at a time.
The issue I was having was that I was using jQuery\'s ajax success hook as I only recently discovered the gem that is ajax.done()!

So my old (simplified!) code looked something like this (notice the _success_ parameter in the _data_ object):
<pre>
function doAjax( posts ) {
	function runProcess( posts ) {
			var current = posts.splice( 0, 10 );
			var data = {
				action: 'my_ajax_action',
				posts: current,
				success: function() {
					runProcess();
				}
			};
			$.post( ajaxurl, data, function() {
				if( allPostsProcessed() ) {
					outputSuccessMessage();
				} else {
					updateProgressMessage();
				}
			} );
		}
	}
	runProcess();
}
</pre>

So this would process 10 of the posts with each request and update the user with messages along the way to notify them of the processes progress. Great! Well, sort of...
The issue was the ajax calls weren\'t running in sequence but all together, they fired one after another but due to the vast number of posts there could be far too many requests running here and on large enough systems server timeouts were getting hit.

To avoid the issue I turned my attention to _.done()_. This was a simple adjustment to the code and ensures that timeouts are avoided by only kicking off another ajax request once the previous has completed.

The code changes were as follows:
<pre>
function doAjax( posts ) {
	function runProcess( posts ) {
			var current = posts.splice( 0, 10 );
			var data = {
				action: 'my_ajax_action',
				posts: current
			};
			$.post( ajaxurl, data, function() {
				if( allPostsProcessed() ) {
					outputSuccessMessage();
				} else {
					updateProgressMessage();
				}
			} ).done( function() {
				runRequest();
			});
		}
	}
	runProcess();
}
</pre>

That seemingly trivial adjustment made all the difference with these processes for me and now meta can be updated without worrying about server limits.

The process itself now takes longer to run due to each request running in turn but for background/admin updates or similar this is not an issue. I just wouldn\'t advise using it on the frontend of the site.