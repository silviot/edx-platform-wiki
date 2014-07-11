This document discusses the design of the Opaque Keys system, as well as the problem Opaque Keys seeks to solve. For practical understanding of how Opaque Keys impacts edX development, see [[Opaque-Keys---Developer-Notes]].

### Table of Contents
1. [Background](#background)
1. [The Problem](#problem)
1. [The Solution](#solution)
  * [Key Introspection API](#introspection)
  * [OpaqueKey Hierarchy](#hierarchy)
  * [Utilizing Keys](#utilizing)
  * [Key Relationships](#relationships)
  * [URLs](#urls)

<a name="background"/>
## Background

Our codebase is in the midst of a database rearchitecture. Our production database currently is referred to as "old Mongo", in preparation for the move to the new architecture known as ["split Mongo"](https://github.com/edx/edx-platform/wiki/Split:-the-versioning,-structure-saving-DAO), but currently all of our production data is stored in old Mongo. The old Mongo database uses keys that contain structured data. These Mongo keys are literally JSON objects, with key-value pairs for `tag`, `org`, `course`, `category`, `name`, and `revision`. XModule provides the `Location` class as an abstraction layer over these Mongo keys. `Location` had a serious deficiency for uniquely identifying xblocks: it does not fully specify the old-style org/course/run identifier (it leaves out the 'run'); thus, old mongo can not uniquely identify xblocks in fully qualified courses. 

Split Mongo uses the full org + course + run but also adds branch and snapshot version (like a git commit hash). The split Mongo project provides the `Locator` class as an abstraction layer over these.

<a name="problem"/>
## The Problem

We pass around serialized Locations and course ids throughout our codebase and allow code to parse these as they see fit. This means that we cannot add fields (such as run, branch, and version) without breaking existing code. As a result, our key abstraction breaks down: rather than merely being a pointer, our application treats these strings as data in their own right, and as a result our application contains all sorts of assumptions and expectations around what data is available, how to modify one key to create a different key, and so on.

This makes it difficult or impossible to effectively reason about how we store our data, and creates nasty ties between the data storage layer and the presentation layer.

One example of this is found in the way that LMS structures its URLs. URL construction is highly dependent on the `org/course/run` triple that is present in our keys, and those three components are parsed separately from the URL. 

This key introspection also means that different parts of our application can only accept `Location`s or only accept `Locator`s, which is odd since both types of keys refer to the same data. It means we must spend time converting from one key abstraction to another, maintaining a `loc_mapper` data structure for the sole purpose of remembering which `Location` refers to which `Locator` and vice versa -- which means lots of extra database queries and performance penalties. It also makes reasoning about how we store our data much more difficult.

<a name="solution"/>
## The Solution

To solve this problem, we convert these "transparent keys" (keys where the application can and does manipulate internal structure and serialized form) into "opaque keys" (keys where the application should be agnostic with respect to key format, and uses an API to get information about the key and to construct new keys). 

The other key benefit of this solution is it allows us to migrate our data from `Location`s to `Locator`s, something we have been trying to do for quite some time. This makes it easier to reason about where and how our data is stored and accessed.

Read on for more about the architecture of OpaqueKeys. For help understanding how to utilize them in your application, see [[Opaque-Keys---Developer-Notes]].

<a name="introspection"/>
### Key Introspection API

Because not all of our application can be easily refactored to treat keys as truly opaque, we have created a key introspection API, `opaque_keys`, that all of our database key abstractions (`Location`s and `Locator`s) support.

The purpose of this API is to allow parts of the application to indirectly introspect database keys, which (a) allows the application to get the information it needs, and (b) ensures that all requests for this information funnel through a very small number of functions. This way, if we need to change the way that the database stores its data, we can do that behind an abstraction layer, and be confident that the rest of the application won't notice. It also means that multiple database key abstractions (`Location`s and `Locators`) can support the same API, so that the rest of the application can treat them as interchangeable, in classic Python duck-typing fashion.  _Thus, key information should only be retrieved using that key's public methods._

<a name="hierarchy"/>
### OpaqueKey Hierarchy

The base abstract Opaque Key class is implemented at `common/lib/opaque_keys/opaque_keys/__init__.py`. There are four main base abstract key classes: `CourseKey`, `DefinitionKey`, `UsageKey`, and `AssetKey`, defined within `common/lib/xmodule/xmodule/modulestore/keys.py`:

                                          OpaqueKey                                         
                    +-------------------+--------------+----------------+                    
                    |                   |              |                |                    
                    |                   |              |                |                    
                    +                   +              +                +                    
                CourseKey          DefinitonKey     UsageKey         AssetKey                
            +--------------+       +----------+    +----------+       +------+                
            |              |       |          +-++-+          |              |                
            +              +       +            ++            +              +                
    SlashSeparated     Course    Definition   Location        BlockUsage     AssetLocation    
      CourseKey         Locator     Locator                      Locator                      
                                                                                        
                                                                 
`CourseKey`s identify courses. A `SlashSeparatedCourseKey` is the course key used for old style `org/course/run` identifiers; a `CourseLocator` is a course key that supports the new style `org+offering` identifier that provides more flexibility in naming conventions.

A `DefinitionKey` is a type of `OpaqueKey` that will be used to identify an XBlock's reusable content.

A `UsageKey` identifies an XBlock usage in a particular context (usually a course). `Location`s and `BlockUsageLocator`s are examples.

A `Location` is both a `DefinitionKey` and a `UsageKey`. `Location`s identify a particular module within a course; they also know about the module's XBlock scope.

An `AssetKey` supports static assets such as pictures, pdfs, and mp3s uploaded by course authors through Studio.

The classes `SlashSeparatedCourseKey` and `Location` both have the `to_deprecated_string` method and `from_deprecated_string` classmethod. This method enables users to serialize and deserialize the CourseKey in the old-style "org/course/run" format and the `Location` in the `i4x://org/course/category/id` format.

<a name="utilizing"/>
### Utilizing Keys

Eventually, no application or interface should specify the type of key needed, only the abstract class (UsageKey, AssetKey,  CourseKey are good, Location and others are bad).

Requiring a Location is a temporary short cut due to LMS not truly treating keys as opaque. We are migrating to a world of true opaqueness, where only the persistence layer (modulestore) should control the concrete key class. Do not write new code that depends on specific types of keys.

<a name="relationships"/>
### Key Relationships

Keys are related in the following way:
                                                                                        
                          CourseKey        DefKey                                             
                          ^       ^                                                     
                          |       |                                                     
                          v       v                                                     
                    AssetKey     UsageKey                                               

A `CourseKey` knows about its `AssetKey`s and `UsageKey`s. An `AssetKey` and a `UsageKey` each knows which `CourseKey` it is associated with. DefKey's are context independent.

<a name="urls"/>
### URLs

We want to have meaningful URLs where possible, which means using slugs instead of numerical IDs or GUIDs. The simple resolution for this is to serialize and deserialize these opaque keys in such a way that the information is not obscured; for example, most `CourseKey`s serialize as `org+course+run` plus possibly other information such as branch and version snapshot id.