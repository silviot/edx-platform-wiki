This document and its children describes how the edx-platform does CRUD (Create, Read, Update, Delete) operations on xblocks and other models and backs those operations with persistence.

Our initial implementation did not distinguish persistence from data access. We're slowly moving to a better division and more clarity of the architectural abstractions, but we have a long way to go. We mix persistence mechanism (e.g., XML, Mongo) with data modeling and operations and furthermore hides some CRUD aspects with xblocks.

For the most part, we currently separate user-relative data from user-independent, content data. These use completely separate persistence mechanisms and transactional semantics. The user-relative data includes not only authentication and authorization but also courseware engagement history and problem submissions. The content data includes the course catalog as well as the course contents.

## User relative data

Currently we use Django's ORM backed by a SQL DB as the persistence layer for user-relative data. We're discussing moving the user history data to a non-SQL db, but the best of all worlds would be for the code to  have no awareness of the persistence mechanism.

## Course catalog and content

Our original course data was in xml stored on a file system. When the server launches, it scans the file system and loads every such course into memory. It then serves the courseware from memory. Any content changes to the course requires restarting the server so it can rescan the data. The `XMLModuleStore` is the data access layer which finds, loads, and serves up the course data to the application. It has only Read access. No Create, Update, nor Delete.

When we launched Studio, we needed a read-write storage mechanism and data access layer.