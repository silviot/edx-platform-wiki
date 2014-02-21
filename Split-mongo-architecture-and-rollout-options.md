# Executive Summary & Action page

**Goal:** how to wire up [split mongo](https://github.com/edx/edx-platform/wiki/Split:-the-versioning,-structure-saving-DAO) asap with as little risk as possible?

**Real goal:** how to get [RESTful api](https://github.com/edx/edx-platform/wiki/RESTful-course-and-xblock-API) asap for SPOCs and other uses? This api depends upon split mongo or at least a full integration of the new Locator syntax.

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

High priority split mongo functionality which is subordinate to the api includes:

1. Flag conflicting edits without losing content (forks)
1. Undo edits
1. Publish transactionally rather than inadvertent dribble

**Risks of going live on split mongo:**

1. Performance: 
  * split creates new versions of courses for every edit. These new versions may significantly expand our storage requirements and may take longer to persist
1. Non-invertability of migration: what to do if a split has a defect since migration is only from old to split?
1. Interruption of in flight or archived courses:
   1. saved bookmarks may not work
   1. hardcoded references may not work (e.g., anchor tags, jumps, etc)
   1. ORA location references need to change to locators
   1. converting student state to reference Locators not Locations
   1. change in address syntax in middle of analytics stream
1. Deciding how to handle uploaded assets: 
  * currently course relative which makes reuse problematic especially for locked assets. 
  * We should decide if we're going to continue to host the assets through the app server and thus should come up with an addressing solution that works on the app server or will we move the assets to a cdn and thus come up with an addressing and locking solution that works in the cdn.
     1. if on app server, should the addressing still be course relative even if multiple courses reference the same assets?

## Phase 1 plan

1. Studio uses Locators, LMS uses Locations
    1. assumes LMS always has the old org/coursenum/run triple which the loc_mapper needs to uniquely map to Locators (if there's more than one course run with the same org/coursenum).
    1. mixed modulestore acts as more than a router and converts all addresses to consumer's format
        1. make mixed wrap each method w/ proper conversion code
        1. change all uses of modulestore through lms and cms to only use mixed not directly go to default, draft, or direct
           1. Change mixed to route to draft if given a Location and revision=='draft'
           1. Change explicit uses of modulestore('draft') to either use a Locator with branch=='draft' or a Location with revision=='draft'
        1. provide 2 mixed modulestore instances: a locator-based one and a location-based one which ensure they treat all calls as providing the declared type and requiring the declared type back on all calls.
            * Convert not only reference fields but also known special indicators inside the data payload like /static, /jump-to, and /jump-to-module (need full enumeration)
        1. to get proper repr to lower level modulestores (Locators to split, Locations to old-mongo) use a config or method on lower level ones which mixed consults and uses to do the conversions in the wrapped methods when sending into the lower level one.
        1. Mixed will look at a local cache to see where the course is. If the cache doesn't say, it will look in split. If it doesn't find it there, it will look in old mongo. When it finds the course, it will record the routing in a memcache to expedite future retrieval.
1. May still need to write top level view handlers for lms which handle any url requests for Locators which somehow slip through the mixed modulestore conversion and convert to Locations, but we believe that any hardcoded Locator references will be authoring errors not things the app should translate.
1. Complete the conversion of Studio to use Locators all the way through (not just in client-server urls, auth, and such places it does now.). For phase 1, don't take advantage of new split functionality.
1. Course migration from old to split mongo will be a controlled dribble: explicitly migrate some subset and increase that subset over time.
      1. Studio needs to support both repositories but only one repository per course; however, migration will not remove the course from old mongo. It just won't update it with on-going edits.
      1. All new courses will go into split.
      1. Migration will be via a command. There will also be a command to roll back to the old mongo version.
      1. At first at least, we will do the migration on a staging server and ask the course team to verify the course before doing the migrations on the production servers. Proposed workflow:
          1. course team, PM, or someone decides to migrate course
          1. send ticket to devops to do migration
          1. devops copies the course to staging (export from prod, import onto staging)
          1. devops invokes the migration command
          1. course team verifies course on staging
          1. course team updates ticket with approval or rejection of migration
          1. devops migrates course on prod
          1. course team verifies course on prod
1. We need to figure out what we're doing with uploaded assets addressing?
  1. Convert also to locators
    1. handle and rewrite old locations
  1. Copy to each reusing course using old Location (c4x)
    1. means supporting old locations in studio just for assets
  1. Move out to the webserver with a new asset specific addressing scheme
    1. handle and rewrite old locations

## Phase 2 to n-1

Some ordering of the following functionality where the studio functionality may only apply to migrated courses. That is, studio may disable the functionality for unmigrated courses.

* studio supports only deliberate publishing as a large transaction not inadvertent publishing of small changes
* studio supports coursewide and xblock undo and redo
* studio supports version comparison on an xblock-by-xblock basis including "use that one" repointing
* studio supports course creation starting from some version of another course (new course points to same snapshot as old course not to a disconnected copy)
* studio supports looking at revisions of related courses (derived from this course, this course derived from, etc)
* studio supports governance around sharing of content (anyone can use my content, only my university system can use my content, only my university, only my department, these specific people or institutions, only my course team, ...)
* studio supports versioning of uploaded assets with all of the above comparisons
* all courses are migrated to split
* analytics supports locators w/ versions
* ora supports locators (or is agnostic)
* lms supports locators (or is agnostic)

## Phase n (end goal)

LMS is Locator based. There is no more location mapping. student, ora, and analytic records version information. ORA and Student table have extra column for Locator and that's what's populated.

**Punchlist for go-live:**

1. Ensure PMs agree to the migration plan and their role. (Don to talk to Jennifer A)
1. xml export from split
1. xml import to split
1. mixed modulestore figure out whether to read & write to split v old mongo v xml
  1. reconcile the method signatures or have mixed know how to invoke each
  1. write loc_mapping calls into mixed as method wrappers
1. command line or admin page to invoke course migration from old to split mongo (unless using lazy migration only)
1. what if any of the split mongo use cases above to support in Studio? What to do w/ that functionality in case of hybrid split b/c old won't support the use cases?
1. hook up Studio to hybrid
1. uploaded assets addressing
1. test, test, test
1. implement the restful api

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

![Pre-split mongo architecture stack](https://raw2.github.com/edx/edx-platform/master/docs/en_us/architecture/presplit.jpg)

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
  1. **Student sees that the course has no content nor outline yet**
1. Teacher creates units and components (Studio)
  1. Studio uses MixedModulestore to create draft entries in Mongo
1. Teacher edits titles and dates for the course, sections, and subsections (Studio)
  1. Studio uses MixedModulestore to update the entries in Mongo
1. Teacher configures grading policy and marks some subsections as graded (Studio)
  1. Studio uses MixedModulestore to update the entries in Mongo
1. Student1 looks at course content (LMS)
  1. LMS uses MixedModulestore to notice that **there is no published content yet for the course**
  1. **Student sees that the course has no content nor outline yet**
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

The focus of this document is the intermediate states we will support as we transition from Location-based to Locator-based. The reason for the intermediate hybrid state is to incrementally deploy functionality and to not require a big bang conversion of all existing course material and records.

Roadblocks to big bang conversion:

1. To enable reusing content among courses and versioning content, the new representation has a richer and slightly incompatible addressing scheme ([Locators](https://github.com/edx/edx-platform/wiki/Locators-and-older-Locations)). This complicates
  1. student state which uses the old locations
  1. analytics using the old locations
  1. references within a course to other course locations
     1. especially if the material is now being referenced in a different course than the original course because the references will reference the original course not the course-invariant address nor the new-course relative address.
  1. similarly references to assets because for some reason they're identified relative to the creating course.
  1. ORA because it uses the Locations to track the lifecycle of each response.
1. Risk around the data migration scripts which have unit tests but which have had no real course content test.
1. The length of time it's taking to finish writing the code for the hypothetical future state.
1. The absence of Studio UX design and development to take advantage of the new functionality (reusing content, undo, comparison, controlled publication, etc)

Strategy for mitigating the risks and roadblocks:

1. To mitigate the addressing schema change,
  1. we've implemented and made live an address scheme mapping service (loc_mapper) which we need to wire wherever needed (it's currently wired at the highest levels of the Studio App and the Studio Client is using the new address scheme).
  1. we decided to temporarily not use the new address scheme in lms so that student state, analytics, grading, drupal, and other such things won't need to be aware of the new address scheme.
  1. we'll use loc_mapper to generate Location-based asset addresses for now, but this means that each course using the same modules must have its own copy of each asset.
  1. we'll use loc_mapper to try to recode cross-course references (assets and xblocks) into within course relative-references. _n_ Locations map to same Locator. loc_mapper knows how to convert to the Locator and then from the Locator to the Location for any of the _n_ course ids which map to it.
1. to mitigate migration risk,
  1. we'll manually invoke migration on a subset of courses and ensure they work well in practice before migrating the others
  1. this some-migrated-some-not state will require ensuring Studio can write to either back end depending on which repository owns the course or using separate code branches.

### Current Architecture (work-in-progress): 

Location mapping in studio app. Using new Locators in studio client. 

![Location mapping for studio app diagram](https://raw2.github.com/edx/edx-platform/master/docs/en_us/architecture/studio%20mapping.jpg)

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
1. **Make mixed modulestore know whether the caller and whether the lower level persistence layer wants Locators or Locations and convert all references on the way in and out to the appropriate type.**

The architecture becomes the following where most of the location mapping is done at the modulestore layer and only inadvertent references get mapped in the apps. The xblock runtime may need to use the loc_mapper as well.

![Location translation at the modulestore layer](https://raw2.github.com/edx/edx-platform/master/docs/en_us/architecture/locator_ubiquity.jpg)

The cohabiting approaches require that the app (at least Studio) never tries to write to any particular modulestore but lets the mixed modulestore layer figure out the routing. That entails changing all modulestore() and get_modulestore() calls as well as writing router logic in mixed.

Cohabiting makes it difficult to make Studio more version aware so that we can begin implementing conflict management (detection as well as resolution), reuse (e.g., spoc reuse of course, course reuse of modules), version comparison, history tracking (show the user who and when last changed each xblock), etc.

The signature and pattern of the methods for split modulestore's create, update, and delete methods differs from old mongo; so, simultaneous support will require that mixed modulestore mediate that difference and we code the app tier to the superset of functionality with some way of handling attempts to use split functionality on a non-split course.

The upside to a gradual migration is that if the version spamming of split has performance implications, the gradual migration will give us time to implement larger granularity version updates as well as other performance improvements before all authors are affected.

Keeping LMS using only old mongo, keeps LMS from tracking versions and thus reconstructing what the student saw or controlling what the student sees by giving the student a consistent version w/in some courseware scope. As long as the publish action also publishes to split's published branch, we'd be able to reconstruct what the student saw via the edited on dates of the published versions in split.