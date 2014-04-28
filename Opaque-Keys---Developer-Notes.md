Currently a WIP

What you should do in your new glorious opaque keys future.

For the most part, extending the platform should not be substantially different. Now instead of passing around `course_id` strings and `location` strings, we will now be passing around OpaqueKey objects.

It is possible to get information from these objects. For example, if you are given a `course_key`, you can use `course_key.org` to get the organization the course belongs to, much as you could have when the course keys were formatted as `org/course/run`.