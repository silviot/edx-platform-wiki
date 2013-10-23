This is a laundry list of things that are maximally confusing as we approach the point where XBlocks run in edx-platform, but most of the code is still written using XModule concepts.

Old-to-new conversions:

- XModules had .location and class Location.  These are implementation details of the CMS/LMS world, and pure XBlock code can't assume ids are implemented this way.  We need to move away from .location, and toward usage_ids.

Confusing things:

- ParentTracker: this fundamentally is about multiple parents for blocks, but with the usage/definition split, we no longer need to have multiple parents. 
- Lots of places that need an id for a block use `.location.url()`, which will be going away.  What should replace it?
 