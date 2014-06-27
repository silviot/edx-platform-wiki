In preparation for enabling use of the [Split Modulestore](https://github.com/edx/edx-platform/wiki/Split%3A-the-versioning%2C-structure-saving-DAO), we have updated edx-platform so accesses to modulestores in LMS and Studio are now always directed through the `MixedModuleStore`.

The `MixedModuleStore` is a server-wide singleton with a pluggable API where one or more courseware persistence providers can be accessed through a single interface.  Higher layers can call the `MixedModuleStore` with `UsageKeys` or `CourseKeys` and know that their calls will be routed to the correct corresponding underlying store.  

Each server instance configures its `MixedModuleStore` with database access parameters and its preferred ordered list of providers.

## Summary of Changes
### Before
Prior to this work, Studio code had to be cognizant of the various Modulestores and explicitly request `Draft` or `Mongo` depending on the revision and category of the `xBlock` it was operating on.  The diagram below illustrates this.

Additionally, logic for traversing hierarchies when doing CRUD operations on xBlocks used to be done in higher layers (in Studio).  

Also, each server instance had duplicated configuraton settings for the various modulestores.   

<img alt="master" src="git-diagrams/mixed_modulestore_before.png" style="float:right">

### After
Now, all code goes through the common `MixedModuleStore` API, and there is a clearer distinction between modulestore-level logic (hierarchy traversal, handling revisions) and higher-level logic (handling `xBlock` exceptional cases such as for `StaticTab`s).

**Server Configuration Settings**
Since different servers may have differing default preferences for the revisions they require, we have 
introduced a new server configuration setting (`MODULESTORE_BRANCH`) for specifying this preference, with currently supported values being `draft-preferred` (set by Studio) and `published-only` (set by LMS and Preview).

Additionally, we have changed the format for the `MixedModuleStore`

* Changed the configuration setting format so the Stores listed in the Mixed configuration can be ordered by preference, via a list rather than a dict.

More detail about the branch and revision constants can be found in the `ModuleStoreEnum` class.

<img alt="master" src="git-diagrams/mixed_modulestore_after.png" style="float:right">

* API changes:
  * Updated APIs such as has_item, get_parent_location, get_items, and delete_item to take in a revision parameter. Refer to docstrings for more detail.
  * Improvements to the implementation of the following APIs:
    * Publish - so it automatically publishes all the downstream children.
    * Delete - so it does the "right" thing depending on which revision is deleted, etc.
