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
1. Course migration from old to split mongo:
  1. Big bang: migrate all courses or all that may be edited?
  1. Lazy: migrate upon attempt to write to old mongo?
  1. Controlled dribble: explicitly migrate some subset and increase that subset over time
      1. Does Studio need to support unmigrated courses for more than read access? (hybrid split)
      1. Will this strategy only apply to edx or also edge and other sites?
1. Use & extend Studio's existing restful api or implement the more general one we proposed (in the short-run)?


**Punchlist:**

1. xml export from split
1. mixed modulestore figure out whether to source & write to split v old mongo v xml
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
  1. Student just sees an outline of the course with no content but with grading and dates
1. Teacher makes some units (u_0..u_i) public (Studio)
  1. Studio uses MixedModulestore to rename draft entries as non-draft ones in Mongo
1. Student1 looks at course content (LMS)
  1. LMS uses MixedModulestore to access all of the courseware from step 1
  1. Student sees content
1. Student1 works through u_0..u_i (LMS)
  1. LMS records student state via xblock runtime to SQL
1. Teacher edits titles, dates, and subsection order for the course, sections, and subsections (Studio)
  1. Studio uses MixedModulestore to update the entries in Mongo
1. Teacher edits u_0..u_i adding new units u_k..u_l between 0 and i and changing the order of some units (Studio)
  1. Studio uses MixedModulestore to copy non-draft entries into ones marked draft and update the draft entries in Mongo
1. Student2 looks at course content (LMS)
  1. LMS uses MixedModulestore to access the courseware
  1. Student sees content in its new chapter, section, and subsection order but not new unit order and does not see the new units nor the changes to u_0..u_i
1. Student2 works through u_0..u_i (LMS)
  1. LMS records student state via xblock runtime to SQL
1. Teacher "publishes" u_0..u_i including u_k..u_l (Studio)
  1. Studio uses MixedModulestore to overwrite non-draft entries with draft ones in Mongo
1. Student3 looks at course content (LMS)
  1. LMS uses MixedModulestore to access the courseware
  1. Student sees content in its new order all the way down and new content
1. Student3 works through u_0..u_i (LMS)
  1. LMS records student state via xblock runtime to SQL
