In preparation for enabling use of the [Split Modulestore](https://github.com/edx/edx-platform/wiki/Split%3A-the-versioning%2C-structure-saving-DAO), we have updated edx-platform so accesses to modulestores in LMS and Studio are now always directed through the `MixedModuleStore`.

The `MixedModuleStore` is a server-wide singleton with a pluggable API where one or more courseware persistence providers can be accessed through a single interface.  Higher layers can call the `MixedModuleStore` with `UsageKeys` or `CourseKeys` and know that their calls will be routed to the correct corresponding underlying store.  

Each server instance configures its `MixedModuleStore` with database access parameters and its preferred ordered list of providers.

# Summary of Changes
## Before
Prior to this work, Studio code had to be cognizant of the various Modulestores and explicitly request `Draft` or `Mongo` depending on the revision and category of the `xBlock` it was operating on.  The diagram below illustrates this.

Additionally, logic for traversing `xBlock` hierarchies used to be dispersed in Studio higher level code.  

Also, each server instance had duplicated configuraton settings for the various modulestores.   

<img alt="master" src="git-diagrams/mixed_modulestore_before.png" style="float:right">

## After
Now, all code goes through the common `MixedModuleStore` API, and there is a clearer distinction between modulestore-level logic (hierarchy traversal, handling revisions) and higher-level logic (handling `xBlock` exceptions such as for `StaticTab`s).

Since different server instances have different preferences for revisions, we have

<img alt="master" src="git-diagrams/mixed_modulestore_after.png" style="float:right">

## MixedModuleStore API
* Updated Studio and LMS so all callers, with a few exceptions, go through the MixedModulestore.
  * to find the exceptions (a few tests and 2 management commands), search for callers to _get_modulestore_by_type.
* Updated the MixedModuleStore API and all their callers to take in the user_id for all Edit operations.

## Mongo Modulestore

* Callers should no longer distinguish between 'direct' and 'draft' Mongo modulestores. All users of the Mongo store now always go through the DraftModuleStore.
* Callers can specify whether they want only published versions in their runtime by configuring the modulestore's branch to 'published' (LMS). The default setting is 'draft' (used by CMS and Preview) which prefers Draft versions over Published versions.
* Since (most) test code now go through the Draft store, some tests needed to be updated to explicitly Publish their newly created items if they assumed they would be in that state.
* API changes:
  * Updated APIs such as has_item, get_parent_location, get_items, and delete_item to take in a revision parameter. Refer to docstrings for more detail.
  * Improvements to the implementation of the following APIs:
    * Publish - so it automatically publishes all the downstream children.
    * Delete - so it does the "right" thing depending on which revision is deleted, etc.

## Settings
* Changed the configuration setting format so the Stores listed in the Mixed configuration can be ordered by preference, via a list rather than a dict.
* Updated the settings for all Test code so they should all go through Mixed, unless they are explicitly testing a particular store.
* Introduced a new Setting called MODULESTORE_BRANCH that specifies the default "branch" (draft or published) to be used by relevant modulestore methods. Currently, the DraftModuleStore uses this setting. Eventually, the SplitModuleStore will also honor it.

