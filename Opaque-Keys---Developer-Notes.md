## The Quick Version

We used to pass around `course_id`s and `location`s as strings.  Now, we are passing these values around as OpaqueKey objects.

For things that were formerly `course_id`s, we now use CourseKeys.

For things that were formerly `locations`, we now use UsageKeys.

## Constructing Opaque Keys

<bold>**This should only be done in tests**.  Avoid explicitly constructing opaque key types in application code.</b>

The best way to construct an opaque key is to use the correct constructor for the correct type of opaque key.

For example:
```
course_key = SlashSeparatedCourseKey('org', 'course', 'run')  # Old-style identifiers
course_key = CourseLocator(org='mit.eecs', offering='6002x', branch = 'published')
usage_key = Location('org', 'course', 'run', 'category', 'name', 'revision')
```

See the [OpaqueKey hierarchy](https://github.com/edx/edx-platform/wiki/Opaque-Keys#opaquekey-hierarchy) to understand what types of keys are available.

### Dealing with old-style serialized data (LMS only)

As of right now, the use of opaque keys (especially on the LMS) is mostly reserved for in-memory representations. While we are in the process of trying to update and migrate old data correctly, you might need to construct an opaque key out of an old-style string-representation of the `course_id` or `location`.

Old-style course_ids have the format `"org/course/run"`.  Old-style locations have the format `"i4x://org/course/category/name`.

For constructing course keys:
```
course_key = SlashSeparatedCourseKey.from_deprecated_string('org/course/run')
```

For constructing locations/usage keys from old-style `i4x` strings (where `course_key` is a `CourseKey` for the course that the location is within):
```
usage_key = course_key.make_usage_key_from_deprecated_string('i4x://org/course/category/name')
```

Studio does not use the `from_deprecated_string` function.

## Getting information out of Opaque Keys

It is possible to get information from these objects. For example, if you are given a `course_key`, you can use `course_key.org` to get the organization the course belongs to. The specific pieces of information that can be retrieved from the keys is dependent on the type of key. Check the implementation of the key to see what pieces of information are available; you can see the docs INCLUDE LINK HERE.

## Serializing Opaque Keys

### Database fields
In many places, we serialize out the `location` or `course_id` to the database. In the past, when these were strings, we used straight `CharField`s to write out the data.

Now that these keys are opaque, we have a few specialized Django Fields written to handle the serialization/deserialization in the database automatically: `CourseKeyField` and `LocationKeyField`. The implementation of these fields can be found in `common/djangoapps/xmodule_django/models.py`

Retrieving the value of one of these fields will give you an opaque key. Trying to assign something other than a `CourseKey` to a `CourseKeyField` or a `Location` to a `LocationKeyField` will cause a validation error.

## Successfully using Opaque Keys

### URL Reverse calls

Reverse calls with new URLs and serialization of keys

OLD:

    course_locator.url_reverse('course/', ''),

NEW (Studio):

    course_url = reverse(
         'contentstore.views.course_handler',
         kwargs={'course_key_string': unicode(course.id)}
    )

NEW (LMS):

    course_url = reverse(
         'instructor.views.instructor_dashboard',
         kwargs={'course_key_string': course.id.to_deprecated_string()}
    )


### Deserializing keys from view handlers

In CMS:

    usage_key = UsageKey.from_string(usage_key_string)

    course_key = CourseKey.from_string(course_key_string)

in LMS:

    usage_key = UsageKey.from_deprecated_string(usage_key_string)

For LMS or other applications reading keys without namespace tags (pre opaque urls), use [[lms deserialization|dealing-with-old-style-serialized-data-lms-only]]

### Pass in opaque keys in view handlers

Remove no longer needed parameters in handlers and methods.  For example:

OLD:

    def course_handler(request, tag=None, org=None, offering=None, branch=None, version_guid=None, block=None):

NEW:

    def course_handler(request, course_key_string=None):

### Remove loc_mapper calls

`loc_mapper` should no longer be used by any CMS or LMS code, except where code is *explicitly* dealing with the Split Mongo store. The application layer has no need to access `loc_mapper`.

* remove from top import statements
* delete all calls

### Create usage keys when needed
OLD:

    handouts_old_location = course_module.location.replace(category='course_info', name='handouts')
    handouts_locator = loc_mapper().translate_location(handouts_old_location, False, True)

or

    handouts_locator = BlockUsageLocator(
          course_key=updates_locator.course_key.version_agnostic(), block_id=block
    )

NEW:

    handouts_locator = course_key.make_usage_key('course_info', 'handouts')

### Creating asset keys (Studio)

    course_key = CourseKey.from_string('org/class/run')
    asset_key = course_key.make_asset_key('asset', 'my_file_name.jpg')

    thumbnail_key = course_key.make_asset_key('thumbnail', 'my_thumbnail_file_name.jpg')

Note that `AssetKey`s only support two `asset_type`s: `'asset'`, which is the asset itself, and `'thumbnail'`, a thumbnail version of the asset.

### Location Transformations

OLD: 

    MITx.999.Robot_Super_Course/branch/draft/block/Robot_Super_Course

NEW: 

    location:MITx+999+Robot_Super_Course+course+Robot_Super_Course
 
Don't call `Location`, but instead use `course_key.make_usage_key`

And NEVER EVER call `Location` with an array, dict, or tuple, and never ever call it w/ `'i4x'` or `'c4x'` prefixes.

## xblock usages of Opaque Keys

The "children" field of an xblock should now contain UsageKeys instead of strings.

The "Reference" type fields (that refer to content defined elsewhere in the course) should also use UsageKeys instead of strings.

## Further Reading

[Opaque Keys Overview](https://github.com/edx/edx-platform/wiki/Opaque-Keys)

[Split Mongo Architecture](https://github.com/edx/edx-platform/wiki/Split:-the-versioning,-structure-saving-DAO).