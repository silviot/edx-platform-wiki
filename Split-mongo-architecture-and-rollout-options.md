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


**Punchlist:**
1. xml export from split
1. mixed modulestore figure out whether to source & write to split v old mongo v xml
1. command line or admin page to invoke course migration from old to split mongo (unless using lazy migration only)
1. what if any of the split mongo use cases above to support in Studio? What to do w/ that functionality in case of hybrid split b/c old won't support the use cases?
1. hook up Studio to split &/or hybrid
1. hook up lms to hybrid
1. test, test, test

#