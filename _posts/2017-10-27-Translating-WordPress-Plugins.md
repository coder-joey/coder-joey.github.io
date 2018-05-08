---
layout: post
title: Translating a WordPress Plugin
---

To add translations and ensure nothing gets overwritten on a plugin update WordPress now provides us with an easy method. Within the wp-content folder one can now add a plugins and/or themes directory, if they don’t already exist. These folders will contain the completed translations for any plugins/themes required and they will not be altered when the extensions are updated.

To translate a plugin we can use Poedit (the free version works just fine). The steps below should be all that are required to generate a shiny new .po and .mo file:

1. Within Poedit, Click File > New from POT/PO file…
2. Choose your translation language
3. Translate each string using either the bottom boxes to enter your own translations or the recommendations in the sidebar provided by Poedit
4. Click File > Save, change the filename to reflect your plugin and adding the country code at the end of the filename, this is important! (e.g. woocommerce-fr_FR.po)
5. If a .mo file is not automatically generated when the .po file is saved you can do this by clicking File > Compile to MO. This should be named the same as your .po file
6. FTP these files into the directory required as shown above. In this examplke we have translated the plugin WooCommerce, therefore the once uploaded to the appropriate directory the filepaths will be wp-content/languages/plugins/woocommerce-fr_FR.po and wp-content/languages/plugins/woocommerce-fr_FR.mo
7. Change the language on your WordPress site in the settings and your translations should appear like magic

One thing to notice (with WooCommerce at least) is that there can be a delay on the translation appearing. It seems WooCommerce specifically handles some translation features itself and so doesn’t instantly display the update. For other plugins tested this worked immediately, however.