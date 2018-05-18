---
layout: post
title: Acceptance Testing in WordPress Plugins
---

Acceptance tests allow us to check that all functionality within our plugin works as it should without the need for a manual process.
This may take a little time to first get into place but being able to run a couple of commands and leave the tests running is an amazing time saver.

Think pre-release, you need to check your plugin is compatible with the last few versions of WordPress and another plugin (in our case WooCommerce). 
We have approximately 50 tests we were manually performing over multiple versions of each plugin - a very, very time consuming, monotonous process.

We had a look at some different options but settled on [codeception](https://codeception.com/) in the end as the documentation is great, it offers a wide variety of options and the setup seemed less of a headache than others.
There is plenty of information out there about spinning this up for any software you want to write tests for so this is just a quick look at my setup and what it allows me to do.

Within my plugin I have an aptly named __tests__ directory which has the following composer.json to pull in everything I need:
<pre>
{
    "require-dev": {
        "lucatume/wp-browser": "~1.21",
        "codeception/codeception": "2.*",
        "phpunit/phpunit": "6.*"
    },
    "scripts": {
        "post-install-cmd": [
            "./run.sh"
        ],
        "post-update-cmd": [
            "./run.sh"
        ]
    }
}
</pre>
At the time of writing, the latest versions of Codeception, PHPUnit and PHPUnitWrapper dont play nice together so I have my PHPUnit version rolled back a little bit until this is patched.

I also have a little script which just pulls in the current version of ChromeDriver and moves it to my vendor/bin directory (this will be created in the next step!) and runs ChromeDriver (not yet needed and will be mentioned later!)
My `run.sh` script looks like this (depending on your system and current version you can alter the URL as required):
<pre>
#!/bin/bash
if [ ! -f vendor/bin/chromedriver ]; then
    echo "Downloading Chromedriver"
    wget -nv https://chromedriver.storage.googleapis.com/2.38/chromedriver_mac64.zip
    echo "Extracting .zip and removing"
    tar -zxvf chromedriver_mac64.zip
    rm chromedriver_mac64.zip
    echo "Moving chromedriver to vendor/bin"
    mv chromedriver vendor/bin/
fi
vendor/bin/chromedriver --url-base=/wd/hub
exit
</pre>

So, to get the ball rolling simply run a `composer install` from the test directory, the first time may take a little while and the output will give you the run down of what you have installed.
Once this is complete ChromeDriver will be running, this isnt yet required as we need to get our tests into place so go ahead and ctrl+c to exit that (later on when you have some tests in place you can clone your repo, run composer install and be up and running straight away).

To get our tests in place codeception provides us with a nice command `vendor/bin/codecept bootstrap` and like magic our directories are created.
We are going to focus on the acceptance tests here so first thing we need to to is head deeper in to /tests/acceptance.suite.yml where we can configure our acceptance test suite. Mine looks like this:
<pre>
# Codeception Test Suite Configuration
#
# Suite for acceptance tests.
# Perform tests in browser using the WPWebDriver or WPBrowser.
# Use WPDb to set up your initial database fixture.
# If you need both WPWebDriver and WPBrowser tests - create a separate suite.

actor: AcceptanceTester
modules:
    enabled:
        - WPDb
        - WPWebDriver
        - \Helper\Acceptance
    config:
        WPDb:
            dsn: 'mysql:host=127.0.0.1;dbname=local'
            user: 'root'
            password: 'root'
            dump: 'tests/_data/dump.sql'
            populate: true #import the dump before the tests
            cleanup: false #import the dump between tests
            url: 'http://example.local'
            urlReplacement: true #replace the hardcoded dump URL with the one above
            tablePrefix: 'wp_'
        WPBrowser:
            url: 'http://example.local'
            adminUsername: 'admin'
            adminPassword: 'password'
            adminPath: '/wp-admin'
        WPWebDriver:
            url: 'http://example.local'
            adminUsername: 'admin'
            adminPassword: 'password'
            adminPath: '/wp-admin'
            browser: chrome
            restart: true
            window_size: false
            port: 9515
            host: 10.0.2.2
            capabilities:
                browserName: chrome
                chromeOptions:
                    args:
                      - --headless
</pre>

A couple of things to note here:
 - WPDb is the WordPress Database settings, this requires details to the **specific database you are setting up for testing**. If you use your main database it will be overwritten! (I have a dedicated installation where I run my plugin test that is completely controlled so I use the default database as shown above)
 - The dump.sql will be overwrite the database each time you start the tests. The settings for this can be changed above but this enables me to have a controlled base line for my install to start at
 - 9515 is just the default port for the ChromeDriver, this is just set to that to avoid any complications
 - `restart: true` means the browser will restart before **every** test. I found that without this set to true I was seeing issues with tests running on logged in users (even when clearing cookies). This will slow the testing down but it has stabilised mine as they would only pass around 50% of the time before
 - `capabilities` can be altered depending on the browser/driver you are using. Here all I wanted to do was ensure the browser was headless to be able to run it in the background

The only other thing you need to do now is supply a base `dump.sql` database export to use as your default (loaded before the tests are run) to the _data directory.

With all that in place you should be able to go ahead and give it a try.  To do this you will need to ssh into your local environment, navigate to the tests directory (containing your composer.json file) and run the following command:
<pre>
vendor/bin/codecept run acceptance
</pre>

This will run all of the tests found within the acceptance directory (I believe one is placed here by default?) and should give you a clear indication of whether everything is setup right.

So, the tests...the tests are written in a lovely format that is so easy to read (but there is a tonne of stuff you can do [check this out](https://codeception.com/docs/03-AcceptanceTests)).

Here is probably the most basic test class we can write to give you a starting point:
<pre>
<?php
class Example_Cest {
	public function ShopHeaderIsShownTest( AcceptanceTester $I ) {
		$I->amOnPage( '/shop' );
		$I->see( 'Shop' );
	}
}
</pre>
With just this file in the acceptance directory when we run `vendor/bin/codecept run acceptance` our headless browser will boot up, navigate to the shop page and check that the text __Shop__ is displayed.

Of course, this is extremely basic but checkout the documentation for everything you can do. Hopefully, this is clear enough to get things started.
If a test fails a screenshot will be taken and added to the tests/_output directory to ease the troubleshooting.

Remember to add the vendor and tests/_output directories to your .gitignore file!