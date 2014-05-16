When [pull request 2905](https://github.com/edx/edx-platform/pull/2905) is merged to master, the way we handle `course_id`s and `location`s will be changing.  This guide includes everything you need to know about how to handle them.

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
  * [Creating Usage Keys](#create_usage)
  * [Creating Asset Keys](#create_asset)
  * [XBlock Usages of Opaque Keys](#xblock)  
  * [Further Reading](#reading)

<a name="quick"/>
## The Quick Version

We used to pass around `course_id`s and `location`s as strings.  Now, we are passing these values around as OpaqueKey objects.

For things that were formerly `course_id`s, we now use CourseKeys.  For things that were formerly `locations`, we now use UsageKeys.

Given the [serialized form](#serialization) of a `course_id` or `location`, you can then [deserialize](#deserialization) it into an OpaqueKey object, which you can then [introspect for information](#introspect).

Before, you would save `course_id` and `location` strings to the database; now you will [save OpaqueKeys to the database](#database).

<a name="serialization"/>
## Serializing

#### In Studio

To serialize a key of any sort (CourseKey, UsageKey, etc) into a string representation, call `unicode(opaque_key)`, where opaque_key is the key you're wanting to serialize.

#### In LMS

To serialize a CourseKey into a string representation, call `foo.to_deprecated_string()`, where `foo` is your CourseKey.

<a name="deserialization"/>
## Deserializing

#### In Studio

In Studio, calling `FooKey.from_string(bar_string)` will give you a `FooKey` key, where `bar_string` is the serialized version of that key.  Examples of serialized keys: 
````
"edx:org+course.run+branch+foo+version+bar+type+baz+block+id"
"course-locator:$org+$course.$run+branch+$branch+version+$version+type"
"ssck:slashes:$org+$course+$run"
````

#### In LMS

In the LMS, the use of opaque keys is mostly reserved for in-memory representations.  While we are in the process of updating and migrating the old data, you will need to construct opaque keys out of old-style string representations of `course_id`s or `location`s.

Old-style course_ids have the format `"org/course/run"`.  Old-style locations have several different formats: `"i4x://org/course/category/name`, `"c4x://org/course/category/name"`, or `"Org.Course.Run/branch/draft/block/Robot_Super_Course"`.

To construct a course key from an old-style course_id:
```
course_key = SlashSeparatedCourseKey.from_deprecated_string('org/course/run')
```

To construct a UsageKey from an old-style `i4x` string (where `course_key` is a `CourseKey` for the course that the location is within):
```
usage_key = course_key.make_usage_key_from_deprecated_string('i4x://org/course/category/name')
```

In very rare cases you may
usage_key = UsageKey.from_deprecated_string(usage_key_string) BLABLA

<a name="introspect"/>
## Introspecting OpaqueKey Objects

It is possible to get information from these objects. For example, if you are given a `course_key`, you can use `course_key.org` to get the organization the course belongs to. The specific pieces of information that can be retrieved from the keys is dependent on the type of key. Check the implementation of the key to see what pieces of information are available; [you can read about the different types of OpaqueKeys here](https://github.com/edx/edx-platform/wiki/Opaque-Keys).

<a name="database"/>
## Saving to the Database
You may see, in our code, custom Django Fields with the names `CourseKeyField` and `LocationKeyField`.  Retrieving the value of one of these fields will give you an opaque key.  The implementation of these fields can be found in `common/djangoapps/xmodule_django/models.py`.

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

<bold>**This should only be done in tests**.  Avoid explicitly constructing opaque key types in application code.</bold>

To construct an opaque key by hand, use the correct constructor for the correct type of opaque key.

For example:
```python
course_key = SlashSeparatedCourseKey('org', 'course', 'run')  # Old-style identifiers
course_key = CourseLocator(org='mit.eecs', offering='6002x', branch = 'published')
usage_key = Location('org', 'course', 'run', 'category', 'name', 'revision')
```

See the [OpaqueKey hierarchy](https://github.com/edx/edx-platform/wiki/Opaque-Keys#opaquekey-hierarchy) to understand what types of keys are available.

<a name="create_usage"/>
#### Creating Usage Keys

OLD WAY:

````
    handouts_old_location = course_module.location.replace(category='course_info', name='handouts')
    handouts_locator = loc_mapper().translate_location(handouts_old_location, False, True)
````

or

````
    handouts_locator = BlockUsageLocator(
          course_key=updates_locator.course_key.version_agnostic(), block_id=block
    )
````
NEW:
````
    handouts_locator = course_key.make_usage_key('course_info', 'handouts')
````
<a name="create_asset"/>
#### Creating Asset Keys (Studio)

    course_key = CourseKey.from_string('org/class/run')
    asset_key = course_key.make_asset_key('asset', 'my_file_name.jpg')

    thumbnail_key = course_key.make_asset_key('thumbnail', 'my_thumbnail_file_name.jpg')

Note that `AssetKey`s only support two `asset_type`s: `'asset'`, which is the asset itself, and `'thumbnail'`, a thumbnail version of the asset.

<a name="xblock"/>
#### XBlock usages of Opaque Keys

The "children" field of an xblock should now contain UsageKeys instead of strings.

The "Reference" type fields (that refer to content defined elsewhere in the course) should also use UsageKeys instead of strings.

<a name="reading"/>
#### Further Reading
*  [Opaque Keys Overview](https://github.com/edx/edx-platform/wiki/Opaque-Keys)
*  [Split Mongo Architecture](https://github.com/edx/edx-platform/wiki/Split:-the-versioning,-structure-saving-DAO)