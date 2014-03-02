## Ideas for improving ways of installing and managing XBlocks.

* Platform developer support
  * Easily add an external XBlock to a course
  * Easily add a new XBlock repository to Studio
* Course developer support
  * Add a pre-installed XBlock to a course (is "Advanced Modules" the right way to do this?)
  * Get documentation for an XBlock
* Studio administrator
  * Register one or more known XBlocks
  * See a list of all XBlocks installed into Studio
  * Determine if newer versions are available and install them
  * Uninstall an XBlock
  * Hide an XBlock
  * Configuration option to allow course developers to add new XBlocks
* XBlock developer
  * Provide a repository of XBlocks that can be consumed
  * Provide a thumbnail for an XBlock
  * Provide online help for an XBlock

## UX ideas

* Implement a 'list of XBlocks' UI
  * Could base this upon the pageable assets UI in Studio
  * Each XBlock could optionally provide a thumbnail for itself
  * Could show a grid of thumbnails instead of one XBlock per row
    * This would be better when choosing a pre-installed XBlock to add to a course
  * Spreadsheet-style UX can show useful metadata (author, version number, install date etc)

## Technical challenges

* How can XBlock installation be supported in production?
  * Is Studio authorized to execute 'pip install' or equivalent programmatically?
* Can a course developed in Studio automatically install its required XBlocks into LMS?
  * Is there a way to know precisely which version is required of each XBlock?

## Tools to look at (both for UX and for functionality)

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

