This document and its children describes how the edx-platform does CRUD (Create, Read, Update, Delete) operations on xblocks and other models and backs those operations with persistence.

Our initial implementation did not distinguish persistence from data access. We're slowly moving to a better division and more clarity of the architectural abstractions, but we have a long way to go. We mix persistence mechanism (e.g., XML, Mongo) with data modeling and operations and furthermore hides some CRUD aspects with xblocks.

For the most part, we currently separate user-relative data from user-independent, content data. These use completely separate persistence mechanisms and transactional semantics. The user-relative data includes not only authentication and authorization but also courseware engagement history and problem submissions. The content data includes the course catalog as well as the course contents.

## User relative data

Currently we use Django's ORM backed by a SQL DB as the persistence layer for user-relative data. We're discussing moving the user history data to a non-SQL db, but the best of all worlds would be for the code to  have no awareness of the persistence mechanism.

## Course catalog and content

Our original course data was in xml stored on a file system. When the server launches, it scans the file system and loads every such course into memory. It then serves the courseware from memory. Any content changes to the course requires restarting the server so it can rescan the data. The `XMLModuleStore` is the data access layer which finds, loads, and serves up the course data to the application. It has only Read access. No Create, Update, nor Delete.

When we launched Studio, we needed a read-write storage mechanism and data access layer; so, we added `MongoModuleStore` with CRUD methods for `create_xmodule`, `save_xmodule`, `update_(item|metadata|children)`, and `delete_item`. Note, we've convoluted the persistence technology (Mongo) with the DAO CRUD functionality. We also separate update according to low-level nature of the data being updated (metadata v children v content).

`MongoModuleStore` stores each xmodule as a separate document in a non-SQL (Mongo) database. In reality, it could be any non-SQL db; however, the db connection and pymongo usages would need to be abstracted out. It uses the xblock's old style location (tag, org, courseid, type, blockid, `draft` or `None`) as the key. Only leaf blocks (e.g., html, problem, video, discussion) and their immediate parents (usually `vertical`) can be marked as `draft`. All structural changes (add, remove, move child) immediately impact the published course. All changes to non-draft modules immediately effect the published course (e.g., dates, grading policy, default randomization).

`MongoModuleStore` attempts to handle inheritance by loading the whole course on each access in order to compute the effective value for every inheritable attribute. This loading is expensive and there are known errors in this inheritance especially when there are both draft and non-draft versions of the same modules in the course.

We created `SplitMongoModuleStore` to fix the expense and fragility of inheritance. In addition it offers full versioning of all edits, the ability to share the same content and settings between courses, as many named branches of a course or named-subcourse structure as you need (e.g., draft, alpha, stage, honors, live). Once again we've convoluted the persistence technology (Mongo) with the DAO functionality (CRUD operations on xblocks). We're hoping to separate these very soon so that this versioning, structure persisting DAO can sit on any non-SQL db.

This page describes this DAO in some detail: [[Split:-the-versioning,-structure-saving-DAO]]