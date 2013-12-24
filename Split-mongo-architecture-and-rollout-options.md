# Executive Summary & Action page

**Issue:** how to wire up split mongo asap with as little risk as possible?

**Real issue:** how to get RESTful api asap for SPOCs and other uses?

Goal RESTful API use cases:

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

Split mongo use cases whose scope is unknown:

1. Flag conflicting edits (forks)
1. Undo edits
1. Publish transactionally rather than inadvertent dribble

There are 2 RESTful APIs:

1. Studio's existing one
1. The proposed central one

Of the above use case, Studio's existing RESTful api supports:

1. Create new course (but from scratch not from existing)
1. Set dates (any xblock field editing)
1. Create, update, or delete any xblocks
1. Publish subtree

**Risks:**

1. Performance: effect of any data migrations if done lazily
1. Non-invertability of migration: what to do if a migrated course has a defect since migration is only from old to split?

**Decisions/options requiring action:**

1. Have Studio support both back ends at the same time for not only read but also write to enable gradual and deliberate course migration (hybrid split)?
  1. broadcast updates to both to enable reversion to old if needed?
  1. just assign courses to one or the other (split v old)?
1. Course migration from old to split mongo:
  1. Big bang: migrate all courses or all that may be edited?
  1. Lazy: migrate upon attempt to write to old mongo?
  1. Controlled dribble: explicitly migrate some subset and increase that subset over time
      1. Does Studio need to support unmigrated courses for more than read access? (hybrid split)
      1. Will this strategy only apply to edx or also edge and other sites?
1. Use & extend Studio's existing restful api or implement the more general one we proposed (in the short-run)?


**Punchlist:**

1. xml export from split
1. mixed modulestore figure out whether to read & write to split v old mongo v xml
  1. if using broadcast model of updates, implement that.
1. command line or admin page to invoke course migration from old to split mongo (unless using lazy migration only)
1. what if any of the split mongo use cases above to support in Studio? What to do w/ that functionality in case of hybrid split b/c old won't support the use cases?
1. hook up Studio to split &/or hybrid
1. hook up lms to hybrid
1. test, test, test
1. extend the studio api or implement the general one

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

![Pre-split mongo architecture stack](https://raw.github.com/edx/edx-platform/dhm/arch-docs/docs/architecture/presplit.jpg)

This document does not currently fully explain this stack, but some notes on this diagram.

* The top (yellow) are the user facing clients: currently just browser clients.
* The next layer (light green) shows the app layer which is primarily restful and non-restful url handlers with any client models and app logic (e.g., most grading).
* The dark green shows (external) grading and analytics as disconnected services purely as a reminder that these and others like these (e.g., drupal) exist not to show how they use the back end. It would be good to get diagrams of how these plug into the back ends.
* The cyan layer is the data access and modeling layer. It handles figuring out the identities and repositories, serializing and deserializing data, determining authorization, etc.
* The xblock runtime currently is subordinate to the modulestore layer which instantiates it, computes addresses, and feeds it data models. lms writes directly to it for student state data which the runtime then persists directly in SQL; however, all courseware writes go through the modulestore layer.

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
1. to mitigate migration risk,
  1. we'll manually invoke migration on a subset of courses and ensure they work well in practice before migrating the others
  1. this some-migrated-some-not state will require ensuring Studio can write to either back end depending on which repository owns the course.
  1. we _may_ make mixed modulestore broadcast each write to each representation (which means it needs both addressing schemes simultaneously and a strategy for handling old mongo's restrictive capabilities).
