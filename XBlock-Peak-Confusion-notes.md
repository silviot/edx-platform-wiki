This is a laundry list of things that are maximally confusing as we approach the point where XBlocks run in edx-platform, but most of the code is still written using XModule concepts.

Old-to-new conversions:

- XModules has `.location` and `class Location`.  These are implementation details of the CMS/LMS world, and pure XBlock code can't assume ids are implemented this way.  We need to move away from .location, and toward usage_ids.

Confusing things:

- ParentTracker: this deals with multiple parents for blocks, but with the usage/definition split, we no longer need to have multiple parents. 
- Lots of places that need an id for a block use `.location.url()`, which will be going away.  What should replace it?
 
XModule Guidelines:

* Don't import `xmodule.modulestore.django` or `xmodule.contentstore.django` from inside an XModule.
    * Rational: Those libraries are externally facing in order to make it easy for a django project (LMS and Studio) to interface w/ XModules. By importing one into an XModule, you make that XModule dependent on Django, which will make it harder to convert to an XBlock later

* Don't ask about the course in an XModule
    * Rational: XModules (and XBlocks) aren't always going to live inside a course. We might want to embed one directly into a page. As such, they shouldn't be trying to access their containing course to find out information about it.
    * Alternative: A common reason for trying to get to the course object is to allow users to set an attribute on the course to influence all of the blocks of a particular type inside that course. To solve this problem, use inheritance (provided by the LMS and Studio) instead. For instance: Say you want to configure the LTI module to use a shared secret. Add that secret as a field on the LTI module:

        class LTIModule(XModule):
            lti_secret = String(scope=Scope.settings)
            
            def get_html(self):
                ... self.lti_secret ...
            
Then add that same field to the InheritanceMixin (in xmodule/modulestore/inheritance.py):

    class InheritanceMixin(XBlockMixin):
        lti_secret = String(scope=Scope.settings)

Now, in Studio, when you edit course settings, the `lti_secret` will be available for editing, and when you set it on the course, that setting will cascade down to all `LTIModules` contained in that course. But if you wanted to use the `LTIModule` outside of a course, you could configure the `lti_secret` for just that single module.