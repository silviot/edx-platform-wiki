When [pull request 4225](https://github.com/edx/edx-platform/pull/4225) is merged, the way we handle `CourseKey`s and `UsageKey`s will change.  This guide includes everything you need to know about how to handle them.

## Table of Contents

1. [The Quick Version](#quick)  
1. [Serializing](#serialization)  
  * In Studio
  * In LMS
1. [Deserializing](#deserialization)  
  * In Studio
  * In LMS
1. [Introspecting  OpaqueKey Objects](#introspect)  
1. [Saving to the Database](#database)  
1. [Related Changes](#related)
  * URL Reverse Calls
1. [Other Notes](#other_notes)
  * [Constructing Opaque Keys by Hand](#constructing)  
  * [Old Way vs. New Way](#old_vs_new)
  * [XBlock Usages of Opaque Keys](#xblock)  
  * [Further Reading](#reading)

<a name="quick"/>
## The Quick Version

We used to pass around course identifiers as `course_id`s, which were strings.  Then, we passed around course identifiers as `CourseKey`s, which were `OpaqueKey` objects.  Now, we pass around course identifiers as `CourseLocator`s, which are a subclass of `CourseKey`.

Similarly, we used to pass around XBlock identifiers as `location`s, which were strings.  Then, we passed around XBlock identifiers as `UsageKey`s, which were `OpaqueKey` objects.  Now, we pass around course identifiers as `BlockUsageLocator`s, which are a subclass of `UsageKey`.

So, the historic path to our Locator-filled present looks like this:

`course_id` -> `CourseKey` -> `CourseLocator`

`location` -> `UsageKey` -> `BlockUsageLocator`

Given the [serialized form](#serialization) of a `course_id`, `location`, `CourseKey`, or `UsageKey`, you can then [deserialize](#deserialization) it into an `CourseLocator` or `BlockUsageLocator` object, which you can then [introspect for information](#introspect).

WEH Before, you would save `course_id` and `location` strings to the database; now you will [save Locators to the database](#database).

<a name="serialization"/>
## Serializing

#### In Studio

To save a Locator of any sort (CourseKey, UsageKey, etc) into a string representation, call `unicode(locator)`, where `locator` is the key you're wanting to serialize.

#### In LMS

To serialize `OpaqueKey`s into a string representation, call `foo.to_string()`, where `foo` is some type of `OpaqueKey` object (e.g., either `CourseKey`, `UsageKey`, `CourseLocator`, or `BlockUsageLocator`).

<a name="deserialization"/>
## Deserializing

#### In Studio

`CourseLocator.from_string(bar_string)`

Calling `FooLocator.from_string(bar_string)` will give you a `FooLocator` object, where `bar_string` is the serialized version of that key.

Examples of serialized `CourseLocator`s:
````python
"org/course/run"  # A deprecated format from when we used course_ids.
"Org.Course.Run/branch/draft/block/Robot_Super_Course"  # A deprecated format from when we used course_ids.
"ssck:slashes:$org+$course+$run"  # A deprecated format from when we used SlashSeparatedCourseKeys.
"course-locator:$org+$course.$run+branch+$branch+version+$version+type"  # The preferred serialized version of a string.
````

Examples of serialized `BlockUsageLocator`s:
````python
"i4x://org/course/category/name"  # A deprecated format from when we used locations.
"c4x://org/course/category/name"  # Another deprecated format from when we used locations.
"edx:org+course.run+branch+foo+version+bar+type+baz+block+id"  # The preferred serialized form of a string.
```

To construct a course key from an old-style course_id:
```python
course_key = SlashSeparatedCourseKey.from_string('org/course/run')
```

To construct a UsageKey from an old-style `i4x` string (where `course_key` is a `CourseKey` for the course that the location is within):

```python
usage_key = course_key.make_usage_key_from_string('i4x://org/course/category/name')
```

If you have no CourseKey (and no `course_id` that you can use to create a CourseKey), a fallback is the UsageKey.from_deprecated_string() method shown below.  Note that this is not preferred; please use the previous method if there's any way you can access the CourseKey.

````python
usage_key = Location.from_deprecated_string('i4x://org/course/category/name')
````

<a name="introspect"/>
## Introspecting Locator Objects

It is possible to get information from these objects. For example, if you are given a `course_locator`, you can use `course_locator.org` to get the organization the course belongs to. The specific pieces of information that can be retrieved from the keys is dependent on the type of key. Check the implementation of the key to see what pieces of information are available; [you can read about the different types of Locators here](https://github.com/edx/edx-platform/wiki/Opaque-Keys).

<a name="database"/>
## Saving to the Database
You may see, in our code, custom Django Fields with the names `CourseKeyField` and `LocationKeyField`.  Retrieving the value of one of these fields will give you a .  The implementation of these fields can be found in `common/djangoapps/xmodule_django/models.py`.

(The reason for this: in many places, we serialize out the `location` or `course_id` to the database. In the past, when these were strings, we used straight `CharField`s to write out the data.  Now that we're using OpaqueKeys, we use these fields to handle the serialization/deserialization in the database automatically.)

Retrieving the value of one of these fields will give you an opaque key. Trying to assign something other than a `CourseKey` to a `CourseKeyField` or a `Location` to a `LocationKeyField` will cause a validation error.

Use `CourseKeyField`s and `LocationKeyField`s instead of `CharField`s to store those data types.

<a name="related"/>
## Related Changes
#### URL reverse calls

Since we have a different way of passing around course identifiers, we do URL reverse calls differently.

OLD WAY (Studio):

````python
course_locator.url_reverse('course/', ''),
````

NEW WAY (Studio):

````python
course_url = reverse(
     'contentstore.views.course_handler',
     kwargs={'course_key_string': unicode(course.id)}
)
````

OLD WAY (LMS):

````python
course_url = reverse(
    'instructor.views.instructor_dashboard',
     kwargs={'course_key_string': course.id}
)
````

NEW WAY (LMS):

````python
course_url = reverse(
     'instructor.views.instructor_dashboard',
     kwargs={'course_key_string': course.id.to_deprecated_string()}
)
````

<a name="other_notes"/>
## Other Notes

<a name="constructing"/>
#### Constructing Opaque Keys by Hand

<bold>**In general, this should only be done in tests**.  Avoid explicitly constructing opaque key types in application code.</bold>

To construct an opaque key by hand, you can always use the correct constructor for the correct type of opaque key.

For example:
```python
course_key = SlashSeparatedCourseKey('org', 'course', 'run')  # Old-style identifiers
course_key = CourseLocator(org='mit.eecs', offering='6002x', branch = 'published')
usage_key = Location('org', 'course', 'run', 'category', 'name', 'revision')
```

**However, for UsageKeys and AssetKeys, it is generally preferable to use the make_usage_key and make_asset_key methods on CourseKey.**  For example, given a CourseKey course_key, you can make usage_key and asset_key for that course as follows:

````python
usage_key = course_key.make_usage_key('course_info', 'handouts')  # 'course_info' is block_type, 'handouts' is the name
asset_key = course_key.make_asset_key('asset', 'my_file_name.jpg')  # 'asset' is type, 'my_file_name.jpg' is the path
````

Note that `AssetKey`s only support two `asset_type`s: `'asset'`, which is the asset itself, and `'thumbnail'`, a thumbnail version of the asset.

See the [OpaqueKey hierarchy](https://github.com/edx/edx-platform/wiki/Opaque-Keys#opaquekey-hierarchy) to understand what types of keys are available.

<a name="old_vs_new"/>
#### Old Way vs New Way

If you encounter legacy code that seems confusing or wrong, see if you can find that pattern in these "old way" examples, and see the "new way" to write that code:

USAGE KEYS, OLD WAY:

````python
handouts_old_location = course_module.location.replace(category='course_info', name='handouts')
handouts_locator = loc_mapper().translate_location(handouts_old_location, False, True)
````

or

````python
handouts_locator = BlockUsageLocator(
    course_key=updates_locator.course_key.version_agnostic(), block_id=block
)
````
USAGE KEYS, NEW WAY:
````python
handouts_locator = course_key.make_usage_key('course_info', 'handouts')
````

<a name="xblock"/>
#### XBlock usages of Opaque Keys

The "children" field of an XBlock should now contain UsageKeys instead of strings.

The "Reference" type fields (that refer to content defined elsewhere in the course) should also use UsageKeys instead of strings.

`xblock.id` used to return locations.  This has been changed; now, to access an xblock's location, use `xblock.location`.

<a name="reading"/>
#### Further Reading
*  [Opaque Keys Overview](https://github.com/edx/edx-platform/wiki/Opaque-Keys-(Locators))
*  [Split Mongo Architecture](https://github.com/edx/edx-platform/wiki/Split:-the-versioning,-structure-saving-DAO)