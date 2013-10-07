This document describes the split mongostore representation which

* separates course structure from content where each course run (or partial course) can have its own structure and content can have many using structures
* and versions everything

It does not describe the original mongostore representation which combined structure and content and used the key to distinguish draft from published elements.

This document does not describe mongo nor its operations. See http://www.mongodb.org for information on Mongo.

## Representation
[This presentation](https://docs.google.com/presentation/d/1u4EoOPwrsQPdZR5Jp9ny1EnRpPgUwsTIybuydkkt4xg/edit?usp=sharing) on google docs provides an overview which may or may not be useful w/o narration.

The xmodule collections:

* `modulestore.active_versions`: this collection maps the org, course, and run to the current draft and published versions of the course.
* `modulestore.structures`: this collection has one entry per course run and one for the template.
* `modulestore.definitions`: this collection has one entry per "module" or "block" version.

**modulestore.active_versions**: a simple map for dereferencing the correct course from the structures collection. It allows naming courses or structures, giving them addressable branches, and using them to fetch the "correct" version for any given course and branch. No course run will have more than one entry in this collection.

```
{
  "_id" : uniqueid,
  "versions" : { <versionName> : versionGuid, ...}
  "edited_by" : user_id,
  "edited_on" : date (native mongo rep)
}
```

* `_id` is a unique id for finding this course run. It's a location-reference string, like `mit.eng.eecs.6002x.industry.spring2013`. (see [Locators (successor to Locations)](https://edx-wiki.atlassian.net/wiki/pages/viewpage.action?pageId=30868230))
* `versions`: These are references to `modulestore.structures`. A location-reference like `mit.eng.eecs.6002x.industry.spring2013/branch/draft` refers to the value associated with `draft` for this document.

* `versionName` is `draft`, `published`, or another user-defined string.
* `versionGuid` is a system generated globally unique id (hash). It points to the entry in `modulestore.structures`

SplitMongo generates a new structure for each change to a structure: that is, for each move, deletion, node creation, or metadata change it adds a new structure and does not modify the old one. If the change was done using a pointer to a from active_index (i.e., course_id and branch), it will update the active_index entry to point to the new structure. No other pointers to the old structure will update. If the change is done using only a structure pointer or if Split determines that the head pointer has moved and the change is really occurring to an "old" structure, the head will not update but split will create a new structure (perhaps forking the structure's history).

Creating a course (creating a new run of a course or such) will create a new entry in this table with just a `draft` branch and will cause a copy of the corresponding entry in `modulestore.structures`. The entry in `structures` will point to its version parent in the source course.

**modulestore.structures**: the entries in this collection follow this definition:

```
{
  "_id" : course_guid,
  "blocks" : {
    block_guid : {  // the guid is an arbitrary id to represent this node in the course tree
      "fields" : {"children": [ block_guid* ], ...}
      "definition" : definition_guid,
      "category" : "section" | "sequence" | ...
      "edit_info" : {"edited_on" : date, "edited_by" : user, "previous_version": course_guid}
      ...// more guids
    },
  },
  "root" : block_guid,
  "original" : course_guid, // the first version of this course from which all others were derived
  "previous" : course_guid | null, // the previous revision of this course (null if this is the original)
  "edited_by" : user_id,
  "edited_on" : date
}
```

* `blocks`: each block is a node in the course such as the course, a section, a subsection, a unit, or a component. The block ids remain the same over edits (they're not versioned).
* `root`: the true top of the course. Not all nodes without parents are truly roots. Some are orphans or ancillary data (e.g., course_info).
* `course_guid`, `block_guid`, `definition_guid` are not those specific strings but instead some system generated globally unique id.
  * The one which gets passed around and pointed to by urls is the `block_guid`; so, it will be the one the system ensures is readable. Unlike the other guids, this one stays the same over revisions and can even be the same between course runs (although the course run contextualizes it to distinguish its instantiated version).
* `definition` points to the specific revision of the given element in `modulestore.definitions` which this version of the course includes.
* `children` lists the `block_guids` which are the children of this node in the course tree. It's an error if the guid in the `children` list does not occur in the `blocks` dictionary. (This will change shortly to enable structures to include other structures by reference.)
* the rest of `fields` will contain the other scope.settings (scope usage = 1, user = 0) fields.

**modulestore.definitions**: the data associated with each version of each node in the structures. Many courses may point to the same definition or may point to different versions derived from the same original definition.

```
{
  "_id" : guid,
  "fields" : ..,
  "default_settings" : {"display_name":..,..}, // a starting point for new uses of this definition
  "category" : xblocktype, // the xmodule/xblock type such as course, problem, html, video, about
  "edit_info" : {
    "original_version" : guid, // the first kept version of this definition from which all others were derived
    "previous_version" : guid | null, // the previous revision of this definition (null if this is the original)
    "edited_by" : user_id,  // the id of whomever pressed the draft or publish button
    "edited_on" : date
  }
}
```

* `_id`: a guid to uniquely identify the definition.
* `fields` is the json representation of the scope.content (definition = 1) fields used by the xmodule.
* `category` is the xmodule type and used to figure out which xmodule to instantiate.

### Templates
All field defaults will be defined through the xblock field.default mechanism. Templates, otoh, are for representing optional boilerplate usually for examples such as a multiple-choice problem or a video component with the fields all filled in. Templates are stored in yaml files which provide a template name, sorting and filtering information (e.g., requires advanced editor v allows simple editor), and then field: value pairs for setting xblocks' fields upon template selection.

### Import/export
Export should allow the user to select the version of the course to export which can be any of the draft or published versions. At a minimum, the user should choose between draft or published.

Import should import the course as a draft course or whatever branch the user chooses regardless of whether it was exported as a published or draft one. If there's already a draft for the same course, in the best of all worlds, it would have the guid to see if the guid exists in the structures collection, and, if so, just make that the current structure (don't do any actual data changes). If there's no guid or the guid doesn't exist in the structures collection, then we'll need to work out the logic for how to decide what definitions to create v update v point to.

### Location
The purpose of `Locator` is to identify content. That is, to be able to locate content by providing sufficient addressing. Our code needs to locate several types of things and uses several different types of locators for these. These are the types of things we need to address. Some of these can be the same as others, but I wanted to lay them out fairly fine grained here before proposing my distinctions:

1. Courses: an object representing a course as an offering but not any of its content. Used for dashboards and other such navigators. These may specify a version or merely reference the idea of the course's existence.
2. Course structures: the names (and other metadata), identifiers, and children pointers but not definitions for all the blocks in a course or a subtree of a course. Our applications often display contextual, outline, or other such structural information which do not need to include definitions but need to show display names, graded as, and other status info. This document's design makes fetching these a single document fetch; however, if it has to fetch the full course, it will require far more work (getting all definitions too) than the apps need.
3. Blocks (uses of definitions within a version of a course including metadata, pointers to children, and type specific content)
4. Definitions: use independent definitions of content without metadata (and currently w/o pointers to children).
5. Version trees: Fetching the time history portrayal of a definition, course, or block including branching.
6. Collections of courses, definitions, or blocks matching some partial descriptors (e.g., all courses for org x, all definitions of type foo, all blocks in course y of type x, all currently accessible courses (published with startdate < today and enddate > today)).
7. Fetching of courses, blocks, or definitions via "human readable" urls. #6 (partial descriptors) may suffice for this as human readable does not guarantee uniqueness.

Some of these differ not so much in how to address them but in what should be returned. The content should be up to the functions not the addressing scheme. So, I think the addressable things are:

1. Course as in #1 above: usually a specific offering of a course. Often used as a context for the other queries.
2. Blocks (aka usages) as in #3 above: a specific block contextualized in a course
3. Definitions (#4): a specific definition
4. Collections of courses, blocks within a specific course, or definitions matching a partial descriptor</li

#### Course locator (course_loc)
There are 3 ways to locate a course:

1. By its unique id in the `active_versions` collection with an implied or specified selection of draft or published version.
2. By its `org` and `prettyid` in the `active_versions` collection with an implied or specified selection of draft or published version. The server should raise an error on any attempt to use this addressing if it does not uniquely identify an entry in the `active_versions` collection. (For now, I'm only supporting this in the partial locator)
3. By its unique id in the `structures` collection.

#### Block locator (block_loc)
A block locator finds a specific node in a specific version of a course. Thus, it needs a course locator plus a `block_guid`.

#### Definition locator (definition_loc)
Just a `guid`.

#### Partial descriptor collections locators (partial)
In the most general case, and to simplify implementation, these can be any payload passable to mongo for doing the lookup. The specification of which collection to look into can be implied by which lookup function your code calls (get_courses, get_blocks, get_definitions) or we could add it as another property. For now, I will leave this as merely a search string. Thus, to find all courses for org = mitx, `{"org": "mitx"}`. To find all blocks in a course whose display name contains "circuit example", call `get_blocks` with the course locator plus `{"fields.display_name" : /circuit example/i}` (the i makes it case insensitive and is just an example). To find if a definition is used in a course, call get_blocks with the course locator plus `{definition : definition_guid}`. Note, this looks for a specific version of the definition. If you wanted to see if it used any of a set of versions, use `{definition : {"$in" : [definition_guid*]}}`

## Publishing

Publishing is the process of making content which is available on one branch also available on another branch. Usually the source branch is a 'draft' or 'editing' branch and the destination branch is a 'published' or 'live' branch. Sometimes we may want the destination branches to be preview, staging, alpha, beta, A-test, honors, or other variants. The new representation makes arbitrary branches very easy.

The difficulty in publishing is that a common use case is to only publish some parts of the course. For example, publish weeks 1 - 3 but not the quiz in week 3 because it's still being edited. Publishing by itself does not imply that the students can see the material because the LMS uses start dates and other mechanisms to decide whether content should be accessible. The reason that partial publication is a "problem" is that it cannot use the simple expediency of just pointing the destination branch to the same structure version as the source branch. Instead it needs to create a new branch which is a subset of the source branch but using the same identities and version information. However, this poses a problem in the history of the destination branch. Is its predecessor version the source branch or the previous published version?

In the old Mongo, there were only 2 branches: live and draft. All course content above verticals (all nodes 2 or more levels above the leaves) was in both at the same time. The act of publishing was the act of copying verticals or leaf components from 'draft' to live. All changes to elements above verticals immediately impacted both the draft and live course (element crud, moving, attribute setting).

In the new Modulestore, the authoring team can select how many branches to support. They could have one: the live one is the draft one. All changes take effect immediately. They could have dozens (draft, experimental, alpha, staged, honors, live, scaffolded, ...). The could publish from any of these to any of the others although in practice that wouldn't make sense.

Publishing a node from one branch to another must imply:
* ensure all of that node's parents are in the destination branch
* ensure the node is in the destination branch in the same relative order vis-a-vis its published siblings under its parent.
* ensure the fields of the node in the destination are the same as the source

What's subject to opinion and how the design will progress:
* publish the node's children to the destination unless they were explicitly excluded via a blacklist provided to the publish api. (That is, publishing works on subtrees not individual nodes).
* if when the node was previously published it had children which no longer exist in the source, remove those from the destination (unless they are also in the blacklist).
* if the node being published is deleted in the source but exists in the destination, then the semantics are to delete the node from the destination.
* any node deletion implies deleting the whole subtree with that given root.

Subject to opinion but will not go into initial implementation:
* update the fields of all tree ancestors of the node to their values in the source branch. The reason to include this behavior is that many of these fields are "policies" governing the behavior of the node's subtree; however, the publishing api should not worry about policy v local field distinctions, imho.

Because just updating a single node's fields will be a common publish step, there should be a version of publish which merely updates the fields. This publishing function should not only update the regular fields but also the order of children and remove any children which have been deleted. It should not add any children, however. The interesting caveat to this is that moving a node involves adding it to one parent and deleting it from another. The api should support this as a single call and not cause the node to disappear from the destination.

### Common publishing use cases

* Update a given node's fields (e.g., display name, dates, grading policy, or change the order of subnodes)
* Move a node from one parent to another
* Publish a "ready" subtree except for some specific not quite ready subtrees under it


### Publish API

`publish( course_locator_w_source_branch, destination_branch, subtree_list, blacklist, node_list )`

* `course_locator_w_source_branch`: a CourseLocator with either a specific version id or a branch from which publishing will occur.
* `destination_branch`: can be either a branch id (just a string) or a CourseLocator with a branch id. Cannot be a version id as the act of publishing implies updating the head of a branch? It is possible to publish from one course to another; however, the block ids may conflict (the old branch's `Chapter4` may be overwritten to something which has no similarity nor commonality). In this case, the system won't do anything to ensure the resulting tree is truly a tree versus a dag where a node may occur more than once.
* `subtree_list`: subtree root block ids where the root and all of its descendants except those in `blacklist` will be copied from source to destination. Can be simply block ids or Locators. If Locators, it's an error if any reference any structure other than the one referenced by `course_locator_w_source_branch`.
* `blacklist`: the subtree roots not to copy from source to destination. Follows the same rules as `subtree_list` and directly subtracts subtrees from `subtree_list`. Note, if any of these nodes already exist in destination, they will remain in their current state. To delete, you need to include the parent in `subtree_list` and not put the node in `blacklist` thus implying making the node the same in destination as in source (deleted).
* `node_list`: just copy the node's fields from source to destination. As if including the node in `subtree_list` but all of its children in `blacklist`.

All changes will occur as if happening in one transaction. Even if the changes to `destination_branch` result in numerous versions, the head won't update until the system commits all changes. Thus, the LMS users' experience should maintain consistency.

In addition to all explicitly listed nodes, publish will ensure each ancestor exists in destination. It will not update any fields of the ancestors which already existed. However, those which did not exist will get their field values from source.