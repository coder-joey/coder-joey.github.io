---
layout: post
title: Developing Locally with WordPress using PHPStorm and Local by FlyWheel
---

I recently switched from VVV ( Varying Vagrant Vagrants) to Local by FlyWheel (formerly Pressmatic).  I loved VVV but without prior knowledge of any of it’s inner workings it was a rather steep learning curve that caused plenty of frustration when things didn’t work as expected. This usually resulted in a fresh install of VVV rather than ever finding where I had messed up. Enter Local.

With Local, the complicated stuff is all hidden away and I can focus on my work and leave the environment setup alone. Detailed below are a few of the ways I do things to make my workflow a little smoother using these amazing tools.

_Note: before even starting to work in wordpress using PHPStorm; make sure wordpress integration is enabled!_

**Setting up a new install**

Whenever I setup a new local site I stick to the same pattern. I allow Local to create the site first and organise my sites within the default “Local sites” directory but I have a particular method of setting up a site which keeps things simple and fast.  This is assuming you have the content directory for the wordpress site you wish to setup and a database copy.

- Firstly, add a site in local by clicking on the + icon located at the bottom left corner of the interface. (I would like to utilise blueprints here as I like to use certain plugins when developing, like debug bar and query monitor, but unfortunately these do not currently work on my system).  I name this something obvious (usually the url of the live site so example.com becomes example.dev).
- I then SSH into the install (right click the site in Local and click open site SSH) and navigate to the wordpress install directory (cd app/public). From here I can utilise wp-cli to drop the default database (avoid any conflicts on import), import the live database and run a search and replace to convert all URLs over. To do this you can run  wp db drop, wp db import path/to/db.sql, wp search-replace old-url new-url
- Now the database is in place we need the content pulled in. This is simply a case of navigating to the sites wp-content directory and dropping in all that is required from the live site (be careful of mu-plugins as some hosts like WPEngine use certain plugins that won’t play nice on a local setup).
- I also run a query to add a new admin user for myself as I like all my local installs to use the same login details (it makes things simple!). To do this I visit the database in sequel pro (link found in Local) and run the following (replacing databasename with the database name and checking the wp_users table for the next available user ID)

``INSERT INTO 'databasename'.'wp_users' ('ID', 'user_login', 'user_pass', 'user_nicename', 'user_email', 'user_url', 'user_registered', 'user_activation_key', 'user_status', 'display_name') VALUES ('1', 'admin', MD5('password'), 'admin', 'admin@local.com', '', '2011-06-07 00:00:00', '', '0', 'admin');``
``INSERT INTO 'databasename'.'wp_usermeta' ('umeta_id', 'user_id', 'meta_key', 'meta_value') VALUES (NULL, '1', 'wp_capabilities', 'a:1:{s:13:"administrator";s:1:"1";}');``
``INSERT INTO 'databasename'.'wp_usermeta' ('umeta_id', 'user_id', 'meta_key', 'meta_value') VALUES (NULL, '1', 'wp_user_level', '10');``

You should now have an exact copy of your live site on your local machine with your own admin user.

**Easily code for WordPress**

With wordpress integration enabled PHPStorm really is a dream to work with. It’s autocomplete for wordpress functions and hooks should really speed things up or avoid unnecessary trips to the codex. If you do need to visit the codex, however, simply right click on the hook/function you want to look up and you have a link in the context menu.

Another neat tool is the ability to jump around within your code just by shift+left clicking on functions/classes etc. This saves so much time when hunting for the root of problems it is probably the biggest appeal of PHPStorm for me personally.

Lastly, the ability to add the wordpress coding style is now something I’d be lost without. This can be done by visiting PHPStorm’s preferences->editor->code style->PHP and choosing wordpress from the dropdown.  This is a great tool for quickly tidying up any mistakes you have made (missing an indent/space). Simple press cmnd+alt+L and watch your code shuffle into place.

**Getting help with or displaying work to others**

A great feature of Local that wasn’t available with VVV is the ability to provide a link for anyone else to view your local site. This is called “Live Link” in Local and runs through ngrok. Usually it is a lot easier to simply show somebody than explain in words what is happening on a site. This helps immensely when requesting help for debugging a problem or checking that what you have done is correct before moving to a live install. Again, this works out of the box and requires no additional setup.

**Debugging in PHPStorm**

To work efficiently when coding it is vital to be able to debug.  This makes finding variables etc. much easier and allows you to quickly find the root of any issues you are facing.  To do this we need to get our IDE, web browser and local install all talking to each other but this can be done in a few easy steps.

- Local has a handy, one-click feature under utilities for all local sites labelled “Configure PHPStorm and IntelliJ IDEs” just for this.  Once enabled we can see it has automatically added PHP server settings to PHPStorm providing the site’s host, port and path allowing PHPStorm to know where to look when debugging.
- To setup the web browser we require an xebug extension. Using chrome we can use “Xdebug helper”. Simply install and activate the extension, visit your local site in the browser and turn debugging on (click the extension icon and press “Debug”). When the bug goes green, you’re good to go.
- Lastly, we need to make sure PHPStorm is accepting these incoming connections from our web browser. To do this we have to enable “Start listening for debug connections” within the IDE. This can be done using the keymap shift+cmnd+L or by clicking the little telephone icon on the top right. Once enabled the icon will change from a red no entry sign to green waves entering the receiver.

That’s it! All you have to do now is set a break point (left click in the margin next to your code where you want a break point to be set), visit your local site and utilise the abundance of information now at your fingertips in PHPStorms debugger.

There are other features within PHPStorm which speed up development that I use (such as integrated FTP, GIT and PHPUnit) but they are not specific to wordpress and so will talk about them elsewhere.