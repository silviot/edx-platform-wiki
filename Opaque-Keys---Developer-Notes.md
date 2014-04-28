Currently a WIP

What you should do in your new glorious [opaque keys](https://github.com/edx/edx-platform/wiki/Opaque-Keys) future.

For the most part, extending the platform should not be substantially different. Now instead of passing around `course_id` strings and `location` strings, we will now be passing around OpaqueKey objects.

# LMS

## Constructing Opaque Keys

In general, the best way to construct an opaque key is to use the correct constructor for the correct type of opaque key.

For example:
```
course_key = SlashSeparatedCourseKey('org', 'course', 'run')
usage_key = Location('org', 'course', 'run', 'category', 'name', 'revision')
```

### Dealing with old-style serialized data

As of right now, the use of opaque keys (especially on the LMS) is mostly reserved for in-memory representations. While we are in the process of trying to update and migrate old data correctly, you might need to construct an opaque key out of an old-style string-representation of the `course_id` or `location`. This is especially true when it comes to urls or external services that send across the old-style strings. We have a few standard patterns for parsing the old-style strings correctly.

For constructing course keys:
```
course_key = SlashSeparatedCourseKey.from_deprecated_string('org/course/run')
```

For constructing locations/usage keys from old-style `i4x` strings (where `course_key` is a `CourseKey` for the course that the location is within):
```
usage_key = course_key.make_usage_key_from_deprecated_string('i4x://org/course/category/name')
```

*Note*: Try not to use this pattern. Use it only when absolutely necessary.

## Getting information out of Opaque Keys

It is possible to get information from these objects. For example, if you are given a `course_key`, you can use `course_key.org` to get the organization the course belongs to. The specific pieces of information that can be retrieved from the keys is dependent on the type of key. Check the implementation of the key to see what pieces of information are available.

## Serializing Opaque Keys

### Database fields
In many places, we serialize out the `location` or `course_id` to the database. In the past, when these were strings, we used straight `CharField`s to write out the data.

Now that these keys are opaque, we have a few specialized Django Fields written to handle the serialization/deserialization in the database automatically: `CourseKeyField` and `LocationKeyField`. The implementation of these fields can be found in `common/djangoapps/xmodule_django/models.py`

Retrieving the value of one of these fields will give you an opaque key. Trying to assign something other than a `CourseKey` to a `CourseKeyField` or a `Location` to a `LocationKeyField` will cause a validation error.

### Serializing to strings
It is unlikely that you will need to produce old-style strings from these opaque keys in any new development that gets done. You will most likely want to pass around the opaque keys as much as possible. But if you need the serialized strings, this is still possible using `to_deprecated_string`.

For example:
```
old_course_id = course_key.to_deprecated_string()
```