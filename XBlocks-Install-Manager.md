Ideas for improving ways of installing and managing XBlocks.

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

UX ideas
* Implement a 'list of XBlocks' UI
  * Could base this upon the pageable assets UI in Studio
  * Each XBlock could optionally provide a thumbnail for itself
  * Could show a grid of thumbnails instead of one XBlock per row
    * This would be better when choosing a pre-installed XBlock to add to a course
  * Spreadsheet-style UX can show useful metadata (author, version number, install date etc)

Technical challenges
* How can XBlock installation be supported in production?
  * Is Studio authorized to execute 'pip install' or equivalent programmatically?
* Can a course developed in Studio automatically install its required XBlocks into LMS?
  * Is there a way to know precisely which version is required of each XBlock?

Tools to look at
* [Chrome extensions](http://0.tqn.com/d/browsers/1/0/w/p/-/-/chrome-disable-extensions-plugins-2.jpg)
* Firefox add-ons
* Eclipse plugins
* Apple App Store

Future directions
* Provide a plugin framework for all aspects of edX tooling (not everything can be an XBlock)
  * Custom reports
  * Custom import/export tools
  * Custom administrative scripts

