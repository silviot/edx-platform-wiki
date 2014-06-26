In preparation for updating edx-platform to use the [Split Modulestore](https://github.com/edx/edx-platform/wiki/Split%3A-the-versioning%2C-structure-saving-DAO), we have implemented an intermediary step 

## MixedModuleStore API
* Updated Studio and LMS so all callers, with a few exceptions, go through the MixedModulestore.
  * to find the exceptions (a few tests and 2 management commands), search for callers to _get_modulestore_by_type.
* Updated the MixedModuleStore API and all their callers to take in the user_id for all Edit operations.

## Mongo Modulestore

* Callers should no longer distinguish between 'direct' and 'draft' Mongo modulestores. All users of the Mongo store now always go through the DraftModuleStore.
* Callers can specify whether they want only published versions in their runtime by configuring the modulestore's branch to 'published' (LMS). The default setting is 'draft' (used by CMS and Preview) which prefers Draft versions over Published versions.
* Since (most) test code now go through the Draft store, some tests needed to be updated to explicitly Publish their newly created items if they assumed they would be in that state.
* API changes:
  * Updated APIs for has_item, get_parent_locations, get_items, and delete_item to take in a revision parameter. Refer to docstrings for more detail.
  * Improvements to the implementation of the following APIs:
    * Publish - so it automatically publishes all the downstream children.
    * Delete - so it does the "right" thing depending on which revision is deleted, etc.

## Settings
* Changed the configuration setting format so the Stores listed in the Mixed configuration can be ordered by preference, via a list rather than a dict.
* Updated the settings for all Test code so they should all go through Mixed, unless they are explicitly testing a particular store.
* Introduced a new Setting called MODULESTORE_BRANCH that specifies the default "branch" (draft or published) to be used by relevant modulestore methods. Currently, the DraftModuleStore uses this setting. Eventually, the SplitModuleStore will also honor it.

## Before
<img alt="master" src="git-diagrams/mixed_modulestore_before.png" style="float:right">

## After
<img alt="master" src="git-diagrams/mixed_modulestore_after.png" style="float:right">
