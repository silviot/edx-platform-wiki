### Problem statement

EdX code throughout the stack currently treats xblock and course ids as transparent, introspectable, and frequently manipulated addresses with a known set of fields. The applications serialize and deserialize these ids as parsable urls with fixed syntax (org, partial course id, category, block id or org, partial course id, course unique id). This treatment makes it almost impossible to add semantics such as versioning, subordinate organizations, lifecycle snapshots (e.g., draft, review, alpha, beta, published), or course encapsulation to the applications.

The db rearchitecture task known as "[Split Mongo](https://github.com/edx/edx-platform/wiki/Split:-the-versioning,-structure-saving-DAO)" uses versioning, suborgs, lifecycle snapshots and separate the identities for courses and reusable definitions from the identity of xblocks within courses (usages). Xblock also distinguishes usages from definitions; so, this change is not unilateral. LMS has also always informally separated course identity from xblock usage identity; however, without treating the course id as an object. In fact, in our existing system, a Location is not necessarily a unique ID and requires the informal course id to make it unique. [Split makes these distinctions formal](https://github.com/edx/edx-platform/wiki/Locators-and-older-Locations).

We had a plan to have the two id representations exist simultaneously with continuous translation between them as necessary ([Rollout Options](https://github.com/edx/edx-platform/wiki/Split-mongo-architecture-and-rollout-options)); however, this plan raised significant performance questions as well as risks around fragility (possibly not finding some hardcoded ids and translating them to the proper representation for the caller or receiver.)

Discussion with Cale, Rob, Ned, DB, and others drove home the point that only the persistence layer should care about xblock usage and definition ids and only the persistence, auth, and registration layers should care about course ids. The application layers (LMS, CMS, analytics, ORA, Forums) should treat these ids as opaque tokens used to perform data CRUD and reference. The application layers should not parse the ids and should not request semantic information from the ids (e.g., category, org, course_id).

All edX apps currently serialize and deserialize the ids as subfields from urls; thus, to make the apps able to handle any persistence layer's id representations, we will need to change all of the urls and the url parsing. Nevertheless, we must provide deprecated backward compatibility which can still interpret hardcoded and bookmarked urls using the existing syntax.

## Proposed solution

The crux of the proposed solution is that all ids (course, xblock, definition) will be opaque keys to the applications. The persistence, auth, and enrollment modules may introspect these keys as necessary for additional information. If an application needs additional information such as org, course_id, category and just has an id, the application must ask either the mixed modulestore, the specific persistence layer, or the key.

If we implement the informational methods such as 'org' on the keys, then the key should refuse to give the information if the information is not practically free to compute. If, however, we implement these methods on the modulestores, then the caller must treat these as potentially expensive. No application should assume such requests are free: that is, the modulestore may perform db operations or other non-trivial lookups to answer the query. 

Sometimes context will make it clear what behaviors the service will likely support for the id (e.g., course_id for an xblock usage but not for an xblock definition, org for a course_id but not necessarily for a definition, category for an xblock usage or definition but not for a course id) but some services may support behaviors which other services don't support (e.g., version and version history for an xblock usage, list of available lifecycle snapshots such as draft v preview v live). Xblocks will themselves support some of these behaviors and, so, in many cases, it may make more sense for the app to retrieve the xblock given the address and query it.

A corollary of this change is that no application should assume the serialized (url, id, etc) form of an id has any particular parsable fields but only that the id is serialized and deserialized as a single field. THUS, the serialized form will not have any slashes in it so that url parsing can digest the whole id as a single field.

### Proposed classes and behaviors

`Key` common superclass representing an address or id of something.
* `unicode(Key)` produces a string representation which will have no slashes, question marks, nor ampersands but may have spaces and other non url safe characters for the given `Key`. _We should decide why we need this behavior separately from the url behavior_ NOTE, rather than storing a much more restricted stringified version of the key in an html `id` attr, store this string or the url below in the `data-id` or any other `data-xxx` field which allows any string chars unlike html attrs which gravely restrict the char set.
* `Key.url()` produces a url safe string for the key which will have no url tag (that is, no `i4x://`), slashes, ampersands, nor question marks and is safe to use as a field in a url.
* `Key(unicode)` constructs a `Key` from the `unicode`. The following must be true: `Key(unicode(key)) == key`.
* `Key.from_url(url)` constructs a `Key` from the url. The following must be true: `Key.from_url(key.url()) == key`.

`Key` is an abstract class and cannot be instantiated. Keys will have namespaces (type indicators) which Key uses to figure out which concrete subclass to instantiate; however, no application should ever try to interpret the namespace indicator nor the payload.

`Key` concrete classes register themselves and their unique namespace id with the `Key` class. The namespaces, of course, cannot collide. If they do, then `Key` may arbitrarily choose which concrete class it uses. (or should it raise an Exception?)

To do more than just serialize and deserialize a key, apps will need to send keys to either a modulestore or request it of the key which will answer reasonable queries about the keys. Such queries may include `org`, `category`, `course_id` for keys which are not `CourseKey`, `version`, `branch`, etc. Key concrete classes  implement the `Key` class interface and register their namespaces with `Key`.

#### Course identity key classes

`CourseKey` is an abstract subclass of `Key` to represent keys which are course ids. These should add support for the following:
* `course_id` is a property of a `CourseKey` instance which returns the unique id for a given course offering. Note, this is not a `course` xblock but instead the offering to which students register, instructors add staff, and SplitMongo provides indices. A `course_id` is a unicode string which uniquely identifies the course. It must obey the same syntax rules as `Key.url()`.
* `CourseKey.from_course_id(course_id)` is a `CourseKey` constructor which inverts the `course_id` property with the concomitant equality requirements.
* `org`? If we decide that keys should answer semantic questions beyond what they obviously represent, then `org` may be one such property; however, `CourseLocator` instances cannot answer the `org` query. Only `Location` or `CourseId` keys. In addition org is actually ambiguous. `harvard` is an org, but so are `harvard.humanities` and `harvard.humanities.political_science`. 

`CourseId` is a concrete implementation of `CourseKey` which represents edX's traditional org, partial course id, and run id triple. Applications should never assume that a `CourseKey` is or will be a `CourseId` and thus should not depend on these 3 fields having any meaning with respect to a `CourseKey`. NOTE, no application should treat course ids as strings of triples any more. `org` is a reasonable query for these.

`CourseLocator` is the existing `CourseLocator` class. `org` is not a reasonable query against these; however, `version` and `branch` are.

#### Usage id key class

`UsageKey` is an xblock usage id: that is, it identifies a particularly xblock in a particular xblock tree (which may or may have a course as its root). xblocks which usages identify have not only `Scope.content` fields but also `Scope.children` and `Scope.settings` fields. This class adds support for one more property:
* `block_id`: a string id for the xblock which is guaranteed to be unique within the context of its course. It is not invertible by itself.
* `from_course_block_ids(course_id, block_id)`: returns a `UsageKey` such that `UsageKey.from_course_block_ids(key.course_id, key.block_id) == key`
* `category`? Does it make sense to require the services to answer `category` queries given usage keys or to make `category` something apps should get from the xblock instance or modulestore? `BlockUsageLocator` instances do not know the `category` of their xblocks.
* `course_id`? If the usage key was retrieved in the context of a `CourseKey`, it would seem reasonable to be able to retrieve the course_id or `CourseKey` from the usage id. Some usage keys will be to usages not in the context of a course; so, it won't make sense for those (e.g., from an orphaned xblock tree fragment) This property is `None` if the usage was not retrieved as part of a course.

`BlockUsageLocator`: the `LocatorService` defines the existing `BlockUsageLocator` concrete implementation of usage key ids. In addition to the above properties and functions, this supports the following which only the persistence layer should count on.
* `version` which returns a unique id for the version such that any other `BlockUsageLocator` with the same version is guaranteed to exist in the same snapshot at the same time.
* `branch`: if the Usage was retrieved in the context of a course, it may have a `branch` which indicates its lifecycle snapshot (e.g., draft, beta, alpha, stage, preview, live, archive).

`Location`: the `Location` class is a concrete implementation of this key class. Note that its implementation of `block_id` combines the information from the old `category` and `name` fields. `Location` is not a complete id; so, we will require that instantiating a `Location` requires passing a `CourseId` unlike the current system. Given a `CourseId`, a `Location` is unique. The current Mongo system has a bug in that it uses only the `Location` as a key and thus cannot handle courses whose ids vary purely in their "run". We potentially propose to "fix" this error for new xblocks.

##### Fixing Location non-uniqueness

Because `Location` does not encode the whole course id and because our existing Mongo modulestore uses `Location` as the primary key, we cannot have two courses which use the same org and catalog number and vary only by run id. We could fix this bug as part of this refactoring by adding the course run to the `Location` and persisting it as part of the key in new documents in the existing mongo stores.

This refactoring would enable course creators to just use the run id as a distinguishing element among their courses.

This refactoring has some potentially serious risks:
* The `Location` dict is used as the primary key. Primary keys must be unique and immutable. Because they're immutable, we cannot upgrade existing documents' primary keys to include the course run or change the order of the fields in the key.
* We must either rewrite existing documents into new documents to incorporate the run id or ensure we continue to find and update the existing documents (find w/o run id) while writing new documents to have the additional subfield in the id.
* Mongo indexes subdocuments by ordered fields and cannot tolerate the fields being in varying orders; thus, if we add the run id (or course_id), we must add it as the last field unless we re-write every document in the db and re-index the db.
* When fetching documents, we must either have 2 query keys--one with and one w/o the run id--or always leave out the run id and filter on run id in the returned cursor.
* Making the queries more flexible will by necessity make them less performant.
* If we choose to "leave out the run id" on queries, we'll need to query via dotted notation rather than subdocument. That is, `find({'_id': {'tag': .., 'org': .., 'course'...}})` must become `find({'_id.tag': .., '_id.org': .., '_id.course'...})` as the former requires all matching docs to exactly match the subdoc. I believe such queries are less performant but I will check.
* When updating documents, we will need to be very careful that we don't accidentally duplicate the docs by writing them out with run ids if the source doc did not have the run id unless we delete the runless source doc.
* If, instead of the above, we decide to rewrite every document in order to populate the run id, we will need to 
  * ensure we can find the run id for every xblock
  * take edX off-line during the update in order to migrate the db
  * ensure everyone hosting their own dbs knows about this process
  * follow standard db migration testing and rollback patterns.

I've submitted a [stack overflow question](http://stackoverflow.com/questions/22155488/using-an-object-subdocument-with-varying-fields-as-id) to get advice on any other gotchas and issues with respect to changing the document fields of the primary key. I will also try to get advice from 10gen.

#### Definition id key class

`DefinitionKey` is the id of the context independent definition of the xblock's content. That is, it points to the xblock's `Scope.content` fields. Courseware development apps will want to use this to reuse content among courses or even within a course.

`Location`-based systems have no separate definition key. When asked for the definition key for an xblock, it will give back the `Location` which is really a `UsageKey` and is not reusable.

`DefinitionLocator`: `Locator`-based system define `DefinitionLocator` to represent the unique id of context independent definitions. These have no additional properties over the implementation of the `Key` properties and functions.

## Other open issues

1. How should apps specify lifecycle branch such as draft, published, staged, etc when asking for a Key?
1. How should apps represent instance revision addressing and revision conflict errors (since you accessed xblock foo, someone else changed it; so, your change branches it or clobbers it. If it branches it, here's the pointers to the competing branch versions. Now, tell me which one you want me to label as the correct one.)
1. How should apps represent what they want to search for xblocks usages and definitions?

## Stories

* STUD-1352: convert urls for all apps
* STUD-1350: convert lms, cms, etc to introspection service
* STUD-1348: introspection service
* STUD-1343: refactor Locations to include full course id
* STUD-1342: engineering spec
* STUD-1341: requirements analysis