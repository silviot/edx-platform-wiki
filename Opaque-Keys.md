Warning: this is a WIP and discusses as-of-yet unmerged features implemented in https://github.com/edx/edx-platform/pull/2905

This document discusses the design of the Opaque Keys system, as well as the problem Opaque Keys seeks to solve. For practical understanding of how Opaque Keys impacts edX development, see [[Opaque-Keys---Developer-Notes]].

### Background

Our codebase is in the midst of a database rearchitecture. Our production database currently is referred to as "old Mongo", in preparation for the move to the new architecture known as ["split Mongo"](https://github.com/edx/edx-platform/wiki/Split:-the-versioning,-structure-saving-DAO), but currently all of our production data is stored in old Mongo. The old Mongo database uses keys that contain structured data. These Mongo keys are literally JSON objects, with key-value pairs for `tag`, `org`, `course`, `category`, `name`, and `revision`. XModule provides the `Location` class as an abstraction layer over these Mongo keys. `Location` had a serious deficiency for uniquely identifying xblocks: it does not fully specify the course id (it leaves out the 'run'); thus, old mongo can not uniquely identify xblocks in fully qualified courses. 

Split Mongo, by contrast, uses unique IDs for database keys -- some of them are uniquely-created strings, and some are Mongo ObjectIds. The split Mongo project provides the `Locator` class as an abstraction layer over these unique IDs.

Because data can migrate from old Mongo to split Mongo, we also have a `loc_mapper`: a database-backed mapping table to map old-style `Location`s to new-style `Locator`s, and vice-versa.

### The Problem

We pass around Location and Locator objects throughout our codebase, as reference to the actual data in Mongo; this is fine. However, in addition to simply passing these objects around, we also have lots of code that introspects these objects, pulling out various pieces of information like `course`, `org`, and `name`, and recombining them in various ways for various purposes. As a result, our key abstraction breaks down: rather than merely being a pointer, our application treats these keys as data in their own right, and as a result our application contains all sorts of assumptions and expectations around the API that these keys support to access that data, what data is available, how to modify one key to create a different key, and so on.

This makes it difficult or impossible to effectively reason about how we store our data, and creates nasty ties between the data storage layer and the presentation layer.

One example of this is found in the way that LMS structures its URLs. URL construction is highly dependent on the `org/course/run` triple that is present in our keys, and those three components are parsed separately from the URL. Studio, on the other hand, has been using the Locator syntax of `org.course.run/branch/version/block`.

This key introspection also means that different parts of our application can only accept `Location`s or only accept `Locator`s, which is odd since both types of keys refer to the same data. It means we must spend time converting from one key abstraction to another, maintaining a `loc_mapper` data structure for the sole purpose of remembering which `Location` refers to which `Locator` and vice versa -- which means lots of extra database queries and performance penalties. It also makes reasoning about how we store our data much more difficult.

## The Solution

To solve this problem, we will convert these "transparent keys" (keys where the application can and does see the internal structure) into "opaque keys" (keys where the application cannot, or chooses not to, get any information about the internal structure). In effect, the goal is to make a key do one thing and one thing only: point to a specific database record. It cannot provide information directly about the record in any way: no org/course/name, no "draft" vs "direct", nothing. This will restore the traditional database key abstraction, and greatly simplify how we interact with our data.

There is a caveat here: not all of the codebase can be easily refactored to treat keys as totally opaque. We do provide a Key Introspection API, discussed in the next section, to do so. However, we wish to continually move in the direction of treating keys as totally opaque.

Parts of our application that currently introspect the key object will have several options for how they can be refactored:

1. The first and best option is to simply not use any information about the key, but only treat it as a pointer. If the application needs to do a database lookup to get information it needs, that's fine; but it will not get information from the key itself.
2. The second option is to examine the larger scope of the problem, and see if the function's caller has done a database lookup to get the information that the function needs: if so, the code can be refactored so that the function accepts the database object rather than the key object. The database object can and should be introspected, because it contains all the information for that record.
3. The third option is to use the key introspection API, as detailed below.

The other key benefit of this solution is it will allow us to migrate our data from `Location`s to `Locator`s, something we have been trying to do for quite some time. This will make it easier to reason about where and how our data is stored and accessed.

Read on for more about the architecture of OpaqueKeys. For help understanding how to utilize them in your application, see [[Opaque-Keys---Developer-Notes]].

### Key Introspection API

Note: This API is provided to support back-compatibility in our application. In the effort to transition to a fully-opaque world, use of this API is discouraged. Particularly, new features should not use this, and efforts should be made to migrate old features off of a dependency on key introspection.

Because not all of our application can be easily refactored to treat keys as truly opaque, we have created a key introspection API, `opaque_keys`, that all of our database key abstractions (`Location`s and `Locator`s) support.

The purpose of this API is to allow parts of the application to indirectly introspect database keys, which (a) allows the application to get the information it needs, and (b) ensures that all requests for this information funnel through a very small number of functions. This way, if we need to change the way that the database stores its data, we can do that behind an abstraction layer, and be confident that the rest of the application won't notice. It also means that multiple database key abstractions (`Location`s and `Locators`) can support the same API, so that the rest of the application can treat them as interchangeable, in classic Python duck-typing fashion.

## OpaqueKey Hierarchy

The base abstract Opaque Key class is implemented at `common/lib/opaque_keys/opaque_keys/__init__.py`. There are four main base abstract key classes: `CourseKey`, `DefinitionKey`, `UsageKey`, and `AssetKey`, defined within `common/lib/xmodule/xmodule/modulestore/keys.py`:

                                          OpaqueKey                                         
                    +-------------------+--------------+----------------+                    
                    |                   |              |                |                    
                    |                   |              |                |                    
                    +                   +              +                +                    
                CourseKey        DefinitonKey        UsageKey         AssetKey                
            +--------------+       +----------+    +----------+       +------+                
            |              |       |          +-++-+          |              |                
            +              +       +            ++            +              +                
    SlashSeparated     Course    Definition   Location        BlockUsage     AssetLocation    
      CourseKey         Locator     Locator                      Locator                      
                                                                                        
                                                                 
`CourseKey`s identify courses. A `SlashSeparatedCourseKey` is the course key used for old style `org/course/run` identifiers; a `CourseLocator` is a course key that supports the new style `org+offering` identifier that provides more flexibility in naming conventions.

A `DefinitionKey` is a type of `OpaqueKey` that will be used to identify an XBlock scope. Implementation of `DefinitionKey`s is reserved for a later point in the Opaque Key transition.

A `UsageKey` identifies an XBlock usage. `BlockUsageLocator`s identify a block (aka module) situated within a particular course.

A `Location` is both a `DefinitionKey` and a `UsageKey`. `Location`s identify a particular module within a course; they also know about the module's XBlock scope.

An `AssetKey` supports static assets. Typically these are uploaded by course authors through Studio.

The classes `SlashSeparatedCourseKey` and `Location` both have the `toDeprecatedString` method. This method enables users to get the CourseKey in the old-style "course id" format.

### Key Relationships

Keys are related in the following way:
                                                                                        
                          CourseKey                                                     
                          ^       ^                                                     
                          |       |                                                     
                          v       v                                                     
                    AssetKey     UsageKey                                               
                                    +                                                   
                                    |                                                   
                                    v                                                   
                                  DefKey

A `CourseKey` knows about its `AssetKey`s and `UsageKey`s. An `AssetKey` and a `UsageKey` each knows which `CourseKey` it is associated with. Further, given a `UsageKey`, it is possible to obtain its associated `DefinitionKey`.

### URLs

We want to have meaningful URLs where possible, which means using slugs instead of numerical IDs or GUIDs. The simple resolution for this is to serialize and deserialize these opaque keys in such a way that the information is not obscured; for example, a `CourseKey` is serialized as `org+offering` (where `offering` is the complement of `org`; in old-style course ids, this would correspond to `course/run`).


## Completed and Ongoing Work

Completing the full transition to OpaqueKeys will comprise several steps, some of which can be executed concurrently.

1. *Pass Location and course_id together throughout applications.* Currently, there are many functions in our codebase that only take a Location or only take a course_id; this is not enough information to uniquely identify a single record in the database, and so we frequently try to look up one from the other. We need to modify our code to always pass Location and course_id together, even if only one or the other is used; this will make it possible for us to replace Location and course_id with Locator.
2. *Convert URLs to use opaque keys.* Currently, our URLs are defined using difference pieces of information from Location in different parts of the URL. For example, `/courses/org/course/name/gradebook`. We need to change these URLs so that the information can be parsed in one segment using a regular expression, something like `courses+org+offering+gradebook`. In Studio, we have URLs that have a serialized Locator, such as `/assets/org.course.name/branch-name/asset_id` -- we also need to remove the slashes from the Locator ID, so that we end up with something more like `assets+org+offering+branch-name+asset_id`.
3. *Prepare for a Mongo migration.* This will take us from a world where all of our course data is stored in the Location format, to a world where all of our course data is stored in the Locator format. This will involve writing migration scripts, copying our production databases to testing databases, and running the migration scripts on the test data to be sure that the migration will be successful. This will also give us an idea of how long our Mongo database will be unavailable (due to the migration).
4. *Prepare our SQL databases to refer to the new Mongo format.* Our SQL databases have some records that contain serialized Locations. We need to create a table in our SQL database that will map old Locations to new Locators, and prepare our queries to join on this table as necessary.
5. *Switch from Locations to Locators.* Because our application will be using opaque keys, the application layer won't be able to tell the difference. This will allow us to remove the course_id being passed everywhere through our application; a Locator contains all the information in both the Location and the course_id, so only once is necessary. This is the reason why we originally had to refactor our application to pass Location and course_id around together; so that we could ensure that we could replace the two of them with one Locator everywhere. Note that for Studio, this will have to be done in such a way that there is a switch that controls whether it writes data in the old Location format or the new Locator format.
6. *Take down Studio and migrate Mongo.* LMS will be able to continue running while our migration script copies all the course data from the old format to the new format. Because LMS is now using Locators, LMS will transparently start reading from the new format as soon as it exists in Mongo. When we bring Studio back online, we will flip the switch so that it will write new data in the new Locator format.
7. *Check performance, optimize indexes.* Undoubtably, there will be performance implications in the new data format. We'll be able to estimate these performance implications in preparing for the migration, but there are always unknowns. If there are extreme problems, we can rollback to the old data.
8. *Delete the old data.* This can be done at some point down the line, when we're sure that the migration was successful.
