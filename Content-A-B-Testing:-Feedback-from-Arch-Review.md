Feedback after discussion of: https://github.com/edx/edx-platform/blob/6a4a860c4c4d8c8a0f19d0adfde60620aa444ba2/docs/developers/source/experiments.rst

The key piece of feedback that we discussed towards the end of the meeting was
focused on the needs of analytics, and how that informed the design of groups.
In particular, it became clear that the easiest way to actually do the later
analysis of user actions would be to tag those actions with the experiments that
the user is in at the time the action took place. That directed discussion of
how to identify which groups the user is in, and how to make that system flexible
enough to handle later needs. The general conclusion was to re-think grouping as
assignment of tags/metadata to users. In particular, one could imagine the to-be-built
user service providing a read-key/write-key functionality that would allow any XBlock
to tag a user with particular data (such as a group assignment) under a particular
key (such as what the name of the experiment is). This same mechanism would also
cover some of the other use-cases brought up in the above document.

Several key thoughts about this proposed design direction:
* It should be explicit whether a key/tag is global, or is only within a course context
* When the course context is added to the analytics events, it should add the user's course-specific tags as well
* When the users global context is added to analytics events, it should add the user's global tags
* User tags should be exposed to xblocks via the runtime (eventually, as part of a user service capability, once XBlock Runtimes support a notion of capabilities).

