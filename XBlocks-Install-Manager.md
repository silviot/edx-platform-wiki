## Motivation

* As an XBlock developer it is too difficult to install and play with an XBlock today
  * See [deployment documentation](https://github.com/edx/edx-platform/blob/master/docs/en_us/developers/source/xblocks.rst#deploying-your-xblock)
* It is impossible for a course developer to install and use a new XBlock
* There is no registry of available XBlocks to choose from
* It would be great to have a list of edX approved XBlocks
* Can we encourage a community of XBlock developers?

## Ideas for improving ways of installing and managing XBlocks.

* Platform developer support
  * Easily add an external XBlock to a course
  * Easily add a new XBlock repository to Studio
* Course developer support
  * Add a pre-installed XBlock to a course (is "Advanced Modules" the right way to do this?)
  * Get documentation for an XBlock
  * Browse a list of known XBlocks
  * Search for an XBlock that provides a desired capability
* Studio administrator
  * Register one or more known XBlocks
  * See a list of all XBlocks installed into Studio
  * Determine if newer versions are available and install them
  * Uninstall an XBlock
  * Hide an XBlock
  * Configuration option to allow or disallow course developers from adding new XBlocks
  * Block XBlocks that are known to be problematic
* XBlock developer
  * Define XBlock dependencies that will be automatically installed with your XBlock
  * Provide a repository of XBlocks that can be consumed
  * Provide a thumbnail for an XBlock
  * Provide online help for an XBlock
  * Contribute XBlock to a public repository 

## UX ideas

* Implement a 'list of XBlocks' UI
  * Could base this upon the pageable assets UI in Studio
  * Each XBlock could optionally provide a thumbnail for itself
  * Could show a grid of thumbnails instead of one XBlock per row
    * This would be better when choosing a pre-installed XBlock to add to a course
  * Spreadsheet-style UX can show useful metadata (author, version number, install date etc)

## Technical challenges

* Need all functionality to be available without code changes 
* How can XBlock installation be supported in production?
  * Is Studio authorized to execute 'pip install' or equivalent programmatically?
* Can a course developed in Studio automatically install its required XBlocks into LMS?
  * Is there a way to know precisely which version is required of each XBlock?

## Tools to look at (both for UX and for functionality)

* WordPress plugins
  * ![WordPress plugins](http://cdn.sixrevisions.com/0278-02_manage_wordpress_plugin_screen.jpg)
* Chrome extensions
  * ![Chrome extensions](http://0.tqn.com/d/browsers/1/0/w/p/-/-/chrome-disable-extensions-plugins-2.jpg)
* Firefox add-ons
  * ![Fixefox add-ons](http://www.accessfirefox.org/Firefox_Beginners_Guide/Add-ons_Customize/Extensions_Options.png)
* Apple App Stores
  * ![Apple Software Update](http://cdn.cultofmac.com/wp-content/uploads/2012/02/Screen-Shot-2012-02-16-at-12.12.02-PM.jpg)

## Future directions

* Provide a plugin framework for all aspects of edX tooling (not everything can be an XBlock)
  * Custom reports
  * Custom import/export tools
  * Custom administrative scripts

