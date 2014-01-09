# Executive Summary & Action page

**Goal:** how to wire up [split mongo](https://github.com/edx/edx-platform/wiki/Split:-the-versioning,-structure-saving-DAO) asap with as little risk as possible?

**Real goal:** how to get [RESTful api](https://github.com/edx/edx-platform/wiki/RESTful-course-and-xblock-API) asap for SPOCs and other uses?

In particular, @rrubin says he wants to see these RESTful API use cases running:

1. SPOC subsetting
    1. Create new course from existing course
    1. Set dates
    1. Delete some chapters, sections, units, &/or components
    1. Publish
1. SPOC compilation
    1. Create new course from existing course
    1. Add chapter from another course
    1. Set dates
    1. Publish

Note: these do not require any lms functionality (grading, student tracking, randomization). The focus is on xblock CRUD and sharing.

I don't know the priority of the following Split mongo use cases, but they've been my highest priorities:

1. Flag conflicting edits (forks)
1. Undo edits
1. Publish transactionally rather than inadvertent dribble

There are 2 RESTful APIs:

1. Studio's existing one
1. The proposed [central one](https://github.com/edx/edx-platform/wiki/RESTful-course-and-xblock-API).

The proposed central one depends upon split mongo or at least a full integration of the new Locator syntax.

Of the above use case, Studio's existing RESTful api supports:

1. Create new course (but from scratch not from existing)
1. Set dates (any xblock field editing)
1. Create, update, or delete any xblocks
1. Publish subtree

**Risks of going live on split mongo:**

1. Performance: effect of any data migrations if done lazily. That is, will large course migrations block user access during migration.
1. Non-invertability of migration: what to do if a split has a defect since migration is only from old to split?
1. Interruption of in flight or archived courses:
   1. saved bookmarks may not work
   1. hardcoded references may not work (e.g., anchor tags, jumps, etc)
   1. ORA location references need to change to locators
   1. converting student state to reference Locators not Locations
   1. change in address syntax in middle of analytics stream
1. Deciding how to handle uploaded assets: currently course relative which makes reuse problematic

**Decisions/options requiring action:**

1. Choose approach (described in detail in main doc below):
  1. LMS & Studio use only Locators: this is the long-term goal, but unlikely in short term
     1. Requires converting lms, ora, student state, and all hard-coded references
     1. Probably requires:
         1. adding Locator column in student state table and logic which prefers this column but can use the existing if this is empty.
         1. possibly duplicating the loc_map table from mongo into rds to enable fast loc mapping in joins in rds
         1. updating ora's table similarly (add another column)
         1. and all of the below
  1. **Studio uses Locators, LMS uses Locations**
     1. Options for managing this address faceting
        1. all references are really the new Locators but they behave like Locations when asked
            1. requires the least amount of new code but is very complicated
            1. will invoke loc_mapper far more frequently
            1. will require changing a lot of hardcoded uses of Locations which use strings, dicts, tuples, or arrays rather than objects (these have to eventually change to make lms use Locators only)
        1. **mixed modulestore acts as more than a router and converts all addresses to consumer's format**
            1. make mixed wrap each method w/ proper conversion code
            1. change all uses of modulestore through lms and cms to only use mixed not directly go to default, draft, or direct
            1. provide 2 mixed modulestore instances: a locator-based one and a location-based one which ensure they treat all calls as providing the declared type and requiring the declared type back on all calls.
            1. to get proper repr to lower level modulestores (Locators to split, Locations to old-mongo):
               1. **either use a config or method on lower level ones which mixed consults and uses to do the conversions in the wrapped methods when sending into the lower level one**
               1. or have all modulestores accept either repr and do its own call to the mapper for any it receives of the undesired repr
    1. May still need to change Locator and Location to handle the app mistakenly treating as if they were the other for any hardcoded references that slip through. I believe the only possible slip would be in urls or embedded in xml which would be inert as far as Studio's concerned (just presented like any other link) but could trip lms on user click.
        1. Write standin methods for each of the other repr's methods or
        1. **Write top level view handlers for lms which handle these and convert to Locatons** or
        1. Require callers to catch exceptions and do the conversion
    1. Complete the conversion of Studio to use Locators all the way through (not just in client-server urls, auth, and such places it does now.); however, either
        1. don't take advantage of new split functionality (xblock reuse, ability to make multiple runs of same course, undo, version comparisions, deliberate rather than incidental publishing, etc) or
        1. make studio "know" which low-level modulestore it got the course from and disable split functionality for old-mongo courses.
1. Course migration from old to split mongo:
  1. Big bang: migrate all courses or all that may be edited?
     1. simplifies Studio as it can assume all courses are in split and thus take advantage of split functionality
     1. riskiest in terms of system performance, interrupting in flight courses, etc (see above risks)
  1. Lazy: migrate upon attempt to write to old mongo?
     1. Slightly less simple but not much less
     1. Slightly less risky in that fewer courses are converted, but still has same risks.
  1. **Controlled dribble: explicitly migrate some subset and increase that subset over time**
      1. **Studio needs to support unmigrated courses for more than read access (hybrid split)**
      1. Will this strategy only apply to edx or also edge and other sites?
      1. Can we implement this strategy by having two separate code branches and servers? One for old mongo and one for split?
1. Hybrid split if chosen: Have Studio support both back ends at the same time for not only read but also write to enable gradual and deliberate course migration?
  1. broadcast updates to both to enable reversion to old if needed?
  1. **just assign courses to one or the other (split v old)?**
1. Separate code branches and servers?
  1. Much less time to go live: don't need to work out co-habitation which has been the main impediment
  1. Requires updating lms to use split backend or changing publishing to publish to old as well as split
  1. Requires production ops to go back to the same type of dispatching as we were using when we had xml and old mongo running on separate servers. At a minimum, this should be a short-term strategy.
1. Use & extend Studio's existing restful api or **implement the more general one** we proposed (in the short-run)?
1. Uploaded assets addressing?
  1. Convert also to locators or
    1. handle and rewrite old locations
  1. Copy to each reusing course or
    1. means supporting old locations in studio just for assets
  1. Move out to the webserver with a new asset specific addressing scheme
    1. handle and rewrite old locations


**Punchlist for go-live:**

1. xml export from split
1. xml import to split
1. mixed modulestore figure out whether to read & write to split v old mongo v xml
  1. if using broadcast model of updates, implement that.
  1. if using hybrid, reconcile the method signatures or have mixed know how to invoke each
1. command line or admin page to invoke course migration from old to split mongo (unless using lazy migration only)
1. what if any of the split mongo use cases above to support in Studio? What to do w/ that functionality in case of hybrid split b/c old won't support the use cases?
1. hook up Studio to split &/or hybrid
1. hook up lms to hybrid or split
1. test, test, test
1. extend the studio restful api or implement the general one

# Architectural depictions with options

To illustrate the differences among these architectures, I will use a combined studio and lms use case. You may want to imagine what you think the students should see at each point:

1. Teacher creates course, sections, and subsections (Studio)
1. Student1 registers for course (LMS)
1. Student1 looks at course content (LMS)
1. Teacher creates units and components (Studio)
1. Teacher edits titles and dates for the course, sections, and subsections (Studio)
1. Teacher configures grading policy and marks some subsections as graded (Studio)
1. Student1 looks at course content (LMS)
1. Teacher makes some units (u_0..u_i) public (Studio)
1. Student1 looks at course content (LMS)
1. Student1 works through u_0..u_i (LMS)
1. Teacher edits titles, dates, and subsection order for the course, sections, and subsections (Studio)
1. Teacher edits u_0..u_i adding new units u_k..u_l between 0 and i (Studio)
1. Student2 looks at course content (LMS)
1. Student2 works through u_0..u_i (LMS)
1. Teacher "publishes" u_0..u_i including u_k..u_l (Studio)
1. Student3 looks at course content (LMS)
1. Student3 works through u_0..u_i (LMS)

## Pre-split mongo

This section covers how the system worked before split mongo and the location mapper.

### Pre-split architecture stack

![Pre-split mongo architecture stack](https://raw2.github.com/edx/edx-platform/master/docs/architecture/presplit.jpg)

This document does not currently fully explain this stack, but some notes on this diagram.

* The top (yellow) are the user facing clients: currently just browser clients.
* The next layer (light green) shows the app layer which is primarily restful and non-restful url handlers with any client models and app logic (e.g., most grading).
* The dark green shows (external) grading and analytics as disconnected services purely as a reminder that these and others like these (e.g., drupal) exist not to show how they use the back end. It would be good to get diagrams of how these plug into the back ends.
* The cyan layer is the data access and modeling layer. It handles figuring out the identities and repositories, serializing and deserializing data, determining authorization, etc.
* The xblock runtime currently is subordinate to the modulestore layer which instantiates it, computes addresses, and feeds it data models. lms writes directly to it for student state data which the runtime then persists directly in SQL; however, all courseware writes go through the modulestore layer. I believe @Cale envisions the xblock runtime as above or encompassing the modulestore; however, it currently doesn't and it's not obvious to me how it can.

### Use case

1. Teacher creates course, sections, and subsections (Studio)
  1. Studio uses MixedModulestore to create entries in Mongo
1. Student1 registers for course (LMS)
  1. LMS uses auth svcs to create entries in SQL
1. Student1 looks at course content (LMS)
  1. LMS uses MixedModulestore to access all of the courseware from step 1
  1. Student just sees an outline of the course with no content
1. Teacher creates units and components (Studio)
  1. Studio uses MixedModulestore to create draft entries in Mongo
1. Teacher edits titles and dates for the course, sections, and subsections (Studio)
  1. Studio uses MixedModulestore to update the entries in Mongo
1. Teacher configures grading policy and marks some subsections as graded (Studio)
  1. Studio uses MixedModulestore to update the entries in Mongo
1. Student1 looks at course content (LMS)
  1. LMS uses MixedModulestore to access all of the courseware from step 1
  1. Student1 sees an outline of the course with no content but with grading, dates, and new titles
1. Teacher makes some units (u_0..u_i) public (Studio)
  1. Studio uses MixedModulestore to rename draft entries as non-draft ones in Mongo
1. Student1 looks at course content (LMS)
  1. LMS uses MixedModulestore to access all of the courseware from step 1
  1. Student1 sees content
1. Student1 works through u_0..u_i (LMS)
  1. LMS records student state via xblock runtime to SQL
1. Teacher edits titles, dates, and subsection order for the course, sections, and subsections (Studio)
  1. Studio uses MixedModulestore to update the entries in Mongo
1. Teacher edits u_0..u_i adding new units u_k..u_l between 0 and i and changing the order of some units and components (Studio)
  1. Studio uses MixedModulestore to copy non-draft entries into ones marked draft and update the draft entries in Mongo
  1. Studio updates the children of the subsections for the inserts and reorders.
1. Student2 looks at course content (LMS)
  1. LMS uses MixedModulestore to access the courseware
  1. Student2 sees content in its new chapter, section, subsection, and unit order but not new component order and does not see the new units nor the changes to u_0..u_i
1. Student2 works through u_0..u_i (LMS)
  1. LMS records student state via xblock runtime to SQL
1. Teacher "publishes" u_0..u_i including u_k..u_l (Studio)
  1. Studio uses MixedModulestore to convert draft entries to non-draft overwriting the existing non-drafts (and removing the drafts) in Mongo
1. Student3 looks at course content (LMS)
  1. LMS uses MixedModulestore to access the courseware
  1. Student3 sees all content in its new order and material
1. Student3 works through u_0..u_i (LMS)
  1. LMS records student state via xblock runtime to SQL

## Long-term split mongo architecture

Eventually split mongo will completely replace the current mongo; so, the diagram will look just like the above one except that Mongo Modulestore will be Split Mongo Modulestore with its 3 collections giving it the ability to support editing undo, reusing content among courses, tracking changes over time (who and when), adding organizational governance over course id namespaces, and running courses over and over without export, rename, and import. The version tracking, for example, will enable the lms to know that student1 did not see the subsequently inserted material and mark which of u_0..u_i changed since the student saw them so the student can decide whether to check out the changes. It will enable analytics to compare performance before and after a courseware change. It will enable course authors to compare versions.

The eventual split mongo architecture will execute the use case above as follows:

1. Teacher creates course, sections, and subsections (Studio)
  1. Studio uses MixedModulestore to create entries in draft course version in Mongo
1. Student1 registers for course (LMS)
  1. LMS uses auth svcs to create entries in SQL
1. Student1 looks at course content (LMS)
  1. LMS uses MixedModulestore to notice that **there is no published content yet for the course**
  1. Student sees that the course has no content nor outline yet
1. Teacher creates units and components (Studio)
  1. Studio uses MixedModulestore to create draft entries in Mongo
1. Teacher edits titles and dates for the course, sections, and subsections (Studio)
  1. Studio uses MixedModulestore to update the entries in Mongo
1. Teacher configures grading policy and marks some subsections as graded (Studio)
  1. Studio uses MixedModulestore to update the entries in Mongo
1. Student1 looks at course content (LMS)
  1. LMS uses MixedModulestore to notice that **there is no published content yet for the course**
  1. Student sees that the course has no content nor outline yet
1. Teacher publishes some units (u_0..u_i) and their parents (Studio)
  1. Studio uses MixedModulestore to create a published branch and version and then copies the draft entries into it via Mongo
1. Student1 looks at course content (LMS)
  1. LMS uses MixedModulestore to access the courseware
  1. Student sees content
1. Student1 works through u_0..u_i (LMS)
  1. LMS records student state via xblock runtime to SQL
1. Teacher edits titles, dates, and subsection order for the course, sections, and subsections (Studio)
  1. Studio uses MixedModulestore to update the entries in draft branch in Mongo
1. Teacher edits u_0..u_i adding new units u_k..u_l between 0 and i and changing the order of some units (Studio)
  1. Studio uses MixedModulestore to update the entries in draft branch in Mongo
1. Student2 looks at course content (LMS)
  1. LMS uses MixedModulestore to access the courseware
  1. **Student2 sees same content as Student1 saw in the same order**
1. Student2 works through u_0..u_i (LMS)
  1. LMS records student state via xblock runtime to SQL
1. Teacher "publishes" u_0..u_i including u_k..u_l (Studio)
  1. Studio uses MixedModulestore to create a new live version with the changes in being published draft entries in Mongo
1. Student3 looks at course content (LMS)
  1. LMS uses MixedModulestore to access the courseware
  1. **Student3 sees content in its new order all the way down and new content**
1. Student3 works through u_0..u_i (LMS)
  1. LMS records student state via xblock runtime to SQL

## Hybrid intermediate state of split running with old mongo

The focus of this document is which of several intermediate state options should we support. The reason for the intermediate hybrid state is to incrementally deploy functionality and to not require a big bang conversion of all existing course material and records.

We have 2 possible approaches:

1. make a purely split version of studio and lms and run on separate server(s) than the current one
1. make the current one simultaneously support some things in split and some in old mongo

Of these, the first is the easiest but runs the risk of another long-running code branch and the expense of nginx having to route requests based on course id (or lack thereof).

Roadblocks to big bang conversion:

1. To enable reusing content among courses and versioning content, the new representation has a richer and slightly incompatible addressing scheme ([Locators](https://github.com/edx/edx-platform/wiki/Locators-and-older-Locations)). This complicates
  1. student state which uses the old locations
  1. analytics using the old locations
  1. references within a course to other course locations
     1. especially if the material is now being referenced in a different course than the original course because the references will reference the original course not the course-invariant address nor the new-course relative address.
  1. similarly references to assets because for some reason they're identified relative to the creating course.
1. Risk around the data migration scripts which have unit tests but which have had no real course content test.
1. The length of time it's taking to finish writing the code for the hypothetical future state.
1. The absence of Studio UX design and development to take advantage of the new functionality (reusing content, undo, comparison, controlled publication, etc)

Strategy for mitigating the risks and roadblocks:

1. To mitigate the addressing schema change,
  1. we've implemented and made live an address scheme mapping service (loc_mapper) which we need to wire wherever needed (it's currently wired at the highest levels of the Studio App and the Studio Client is using the new address scheme).
  1. we decided to temporarily not use the new address scheme in lms so that student state, analytics, grading, drupal, and other such things won't need to be aware of the new address scheme.
  1. we'll use loc_mapper to generate the asset addresses for now
  1. we'll use loc_mapper to try to recode cross-course references (assets and xblocks) into within course relative-references. _n_ Locations map to same Locator. loc_mapper knows how to convert to the Locator and then from the Locator to the Location for any of the _n_ course ids which map to it.
  1. If we separate the split and old mongo servers, then we don't need as much wiring for addresses. We could begin using the new address scheme in lms, but we'd need to either map to old for analytics, student state, drupal or update those to the new address scheme.
1. to mitigate migration risk,
  1. we'll manually invoke migration on a subset of courses and ensure they work well in practice before migrating the others
  1. this some-migrated-some-not state will require ensuring Studio can write to either back end depending on which repository owns the course or using separate code branches.
  1. we _may_ make mixed modulestore broadcast each write to each representation (which means it needs both addressing schemes simultaneously and a strategy for handling old mongo's restrictive capabilities).

### Current Architecture (work-in-progress): 

Location mapping in studio app. Using new Locators in studio client. 

![Location mapping for studio app diagram](https://raw2.github.com/edx/edx-platform/master/docs/architecture/studio%20mapping.jpg)

The difference here is the insertion of the loc_mapper and its store. The studio app takes each outgoing Location and uses the loc_mapper to convert it to a Locator so that all the client sees are Locators. It takes each incoming Locator and reconverts it back to a Location so that all the MixedModulestore sees is Locations.

This change has no effect on the current use case other than the form of the urls (which the use case does not discuss).

The problem is how to wire split mongo which uses Locators and old mongo and xml which use Locations without having the applications know which of the two addressing schemes the underlying data access and modeling layer uses. This problem is complicated by the fact that addresses are usually passed around merely as strings without any hint to their semantics and often hidden within other structures. Another complication is that those using Locations must also provide the unique id for the course to get a valid Locator. The loc_mapper will give a mapping even if it doesn't know which course is really in effect, but that mapping may be wrong. In practice, we don't allow more than one course with the same org and "course name"; so, most mappings will be correct; however, we cannot guarantee that they will.

The above diagram's depiction of converting at the app tier does not work for using split mongo which does not want the conversion.

Considered approaches<a name="locator-location-locus"/>:

1. use only the new Locators wherever we know that a field is an address instead of using strings, Locations, or other inert types (dicts, tuples, arrays).
  1. add behavior to these Locators for them to mock old Location functionality for read as well as create and write (calling loc_mapper as necessary)
  1. ensure every code place does not merely pass these around as assumed strings or ensure that the objects present such strings wherever such assumptions lie.
1. move the mapping functionality to the low level modulestore methods having them accept any address form and converting it to whatever representation that modulestore needs via the loc_mapper.
  1. to ensure existing higher level code does not trip on alternative representations, we'd have to 
     1. ensure those functions just pass the address around inertly, 
     1. duplicate each xblock field which we know holds addresses and have a version of the field for each repr, 
     1. ensure each access stipulates what type of address it wants (and provides the course_id), or
     1. tell the modulestore which representation to populate into the reference fields according to which app requested it.
1. use separate code branch and servers: 
  1. leave the current as-is and focus on making a split only version
  1. use the current to migrate courses over to the new
  1. in the new, use only Locators no Locations
      1. verify that student state, analytics, and drupal will accept Locators or
      1. convert the Locators to Locations for any outside service which cannot handle Locators or
      1. update the outside services to handle Locators perhaps also on separate branch w/ separate servers?

Of the 2 above approaches, the first and last seem cleanest. The first has some risks including performance because our code frequently calls Location methods, race conditions if the code requests a translation before the loc_mapper knows about the course, and the need to do 2 pass conversions for inadvertent wrong-course hard-coded references (see above where I described asset and in course references for things borrowed by other courses) (this dual conversion problem exists in both approaches). For the second approach, none of the sub-approaches is sufficient in and of themselves. The last (encoding the address according to the application's preference) may be the closest to sufficient; however, because the code will not know how to find each reference in an xblock, some will leak to the upper layers which will need to catch address failures and attempt conversions.

For either approach, we'll need to decide whether to convert the existing mongo (aka, "old mongo") to read and write persisted addresses in either representation or only use Locations because that's what old mongo uses now. For the separate server approach, it will only use the new Locators.

In the long run, I'd like to deprecate the old Location and its behavior; however, it's not clear how we get there.

If we use either mixed address approach, the architecture becomes the following where most of the location mapping is done at the modulestore layer and only inadvertent references get mapped in the apps. The xblock runtime may need to use the loc_mapper as well.

![Location translation at the modulestore layer](https://raw2.github.com/edx/edx-platform/master/docs/architecture/locator_ubiquity.jpg)

### Hybrid approaches

The hybrid approaches for running split mongo alongside or on a separate server from old mongo have several control dimensions:

1. Which courses persist in split and which into old mongo?
  1. All courses: one time big bang conversion--unlikely approach.
  1. All current and future courses: leave archived courses alone but don't allow access from Studio--also unlikely.
  1. Any course being edited in Studio: proactively move any course which should be accessible in Studio and have Studio only use split mongo. (not possible in the separate server implementation)
  1. Lazily any course being edited in Studio: read from either store, but only allow writes to split mongo. This was the approach I was working on. It would force migration from old to split upon first update attempt.
  1. All new courses, but leave old ones in old mongo: this strategy doesn't save any work but may reduce risk for running courses by ensuring that no addresses change. It requires having Studio able to read and write to both stores and having LMS able to read from both (all of the below do as well).
  1. All new courses plus a gradually increasing set of other deliberately migrated courses.
1. Should Studio use split but LMS use old mongo?
  1. Requires writing a publish mechanism from split to old mongo.
  1. Still requires determining strategy for when to move courses to split for Studio.
1. Should Studio broadcast updates to both stores to enable easy roll-back?
  1. Will require some additional work as well as analysis as what information is lost in the old mongo version and whether we care about that loss.

Whatever choice we make is an interim choice; so, we need to patch together a path from all old mongo to all split no matter how hypothetical that end point may be.

#### Comparative effort estimates

##### Locator - Location approaches

Needed for cohabiting approaches, not the dual server approach.

1. Locator w/ Location veneer 
  1. Remove assumptions that Location is a tuple
  1. Change loc_mapper to know that these are not distinct types
  1. Upon instantiation, loc_map each Location to populate the Locator fields.
  1. Upon Location attr access, loc_map each Locator to get its Location fields (map once, cache)
1. Lazy Locator w/ Location veneer. Location is a separate class. Attempts to access Location attrs on a Locator forces mapping and vice versa.
  1. Don't need to change any access patterns
  1. Upon Locator attr access, loc_map each Location to get its Locator fields (map once, cache)
  1. Upon Location attr access, loc_map each Locator to get its Location fields (map once, cache)
1. Make app ignore reference type and have all mapping done at modulestore.
  1. Update all modulestore functions to take either Locator or Location and call loc_mapper if it didn't get the type it expected.
  1. Deserialize ids and children etc from persistence into either Locator or Location according to 
     1. runtime preference?
     1. type info in serialized form?
  1. Have app tier pass Location/Locator around as intact objects rather than fields:
     1. Change each url and view function to accept either set of fields and cons up the appropriate object
     1. Refactor each app tier access of Location/Locator attrs to get via correct pattern or not use attr

##### Hybrid approaches

The cohabiting approaches require that the app (at least Studio) never tries to write to any particular modulestore but lets the mixed modulestore layer figure out the routing. That entails changing all modulestore() and get_modulestore() calls as well as writing router logic in mixed.

Any implementation in which Studio must support both old and split mongo for write access is roughly equivalent in effort and additional effort over the existing planned work (which assumes we're converting all writes to Split). I'm fairly concerned about not only the amount of work for this simultaneous support but also the functional limitations as I was envisioning the app tier quickly becoming more version aware so that we can begin implementing conflict management (detection as well as resolution), reuse (e.g., spoc reuse of course, course reuse of modules), version comparison, history tracking (show the user who and when last changed each xblock), etc.

The signature and pattern of the methods for split modulestore's create, update, and delete methods differs from old mongo; so, simultaneous support will require that mixed modulestore mediate that difference and we code the app tier to the superset of functionality with some way of handling attempts to use split functionality on a non-split course.

The upside to a gradual migration is that if the version spamming of split has performance implications, the gradual migration will give us time to implement larger granularity version updates as well as other performance improvements before all authors are affected.

The additional work for broadcast update over supporting either will mainly impact performance, but it will require a few days of development work at the mixed modulestore layer. 

If split does not have to lazily convert any courses, it does remove that behavior from mixed modulestore which will save some work (not a lot).

Keeping LMS using only old mongo reduces work on LMS but adds the task of either reverse migration from split to old mongo or, better, changing publish from split to support writing to old mongo. In the short-run, the main impact will be flattening out reuse references into copies of reused content in old mongo. In the long-run, it will keep LMS from tracking versions and thus reconstructing what the student saw or controlling what the student sees by giving the student a consistent version w/in some courseware scope. As long as the publish action also publishes to split's published branch, we'd be able to reconstruct what the student saw via the edited on dates of the published versions in split.