This is a laundry list of things that are maximally confusing as we approach the point where XBlocks run in edx-platform, but most of the code is still written using XModule concepts.

Outline of things to change to move down from Peak Confusion:

- Per-block runtimes (created by modulestore, module_render)
- Per-block field datas (created by modulestore, module_render)
    - FieldDatas should know how to read/write to the DB, rather than just having local memory state that something else persists
- Separate runtimes for XModules and XModuleDescriptors
- Ball-of-mud runtimes (runtimes without segregated responsibilities)
- Use of the word `system`
- Conglomeration of serialization (xml import) and storage (in-memory) in the XML Module Store
- Modulestore api (divide responsibilities between FieldData, UsageStore, and separate interface for version management)
- Use of `to_json` and `from_json` for client facing code. 
    - Use `singledispatch` to allow for many serialization targets?
- Use of `handler_prefix` to implement JS `handler_url` code
- `ErrorModules` and `ErrorDescriptors` don't act like the `XModule`s and `XModuleDescriptor`s they replace
    - Use `ErrorMixin` instead that has an `is_error` attribute, and possibly overrides views with debugging info?

Old-to-new conversions:

- XModules has `.location` and `class Location`.  These are implementation details of the CMS/LMS world, and pure XBlock code can't assume ids are implemented this way.  We need to move away from .location, and toward usage_ids.

Confusing things:

- ParentTracker: this deals with multiple parents for blocks, but with the usage/definition split, we no longer need to have multiple parents. 
- Lots of places that need an id for a block use `.location.url()`, which will be going away.  What should replace it?
 
XModule Guidelines:

* Don't import `xmodule.modulestore.django` or `xmodule.contentstore.django` from inside an XModule.
    * Rational: Those libraries are externally facing in order to make it easy for a django project (LMS and Studio) to interface w/ `XModules`. By importing one into an `XModule`, you make that `XModule` dependent on Django, which will make it harder to convert to an `XBlock` later

* If an `XModule` (or `XBlock`) uses the contents of a field, then that field should be defined on that `XModule` (or `XBlock`). Don't rely on the definition of the field in a `Mixin`.
    * Rational: `XBlock`s should be usable without the Inheritance mechanism (they should be stand-alone), so that they can be tested independently.

* Don't ask about the course in an XModule
    * Rational: XModules (and XBlocks) aren't always going to live inside a course. We might want to embed one directly into a page. As such, they shouldn't be trying to access their containing course to find out information about it.
    * Alternative: A common reason for trying to get to the course object is to allow users to set an attribute on the course to influence all of the blocks of a particular type inside that course. To solve this problem, use inheritance (provided by the LMS and Studio) instead.


Inheritance example:

Say you want to configure the LTI module to use a shared secret. Add that secret as a field on the LTI module:

    class LTIModule(XModule):
        lti_secret = String(scope=Scope.settings)
        def get_html(self):
            ... self.lti_secret ...
            
Then add that same field to the InheritanceMixin (in xmodule/modulestore/inheritance.py):

    class InheritanceMixin(XBlockMixin):
        lti_secret = String(scope=Scope.settings)

Now, in Studio, when you edit course settings, the `lti_secret` will be available for editing, and when you set it on the course, that setting will cascade down to all `LTIModules` contained in that course. But if you wanted to use the `LTIModule` outside of a course, you could configure the `lti_secret` for just that single module.