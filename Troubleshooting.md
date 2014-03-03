### LMS

#### When I log in I keep getting redirected to the login page

The symptom here is after login you get bounced right back to the login page, this time with a breadcrumb to the dashboard page.  This can happen when the login endpoint is failing due to a schema issue.  When Django views fail on a server error, in debug mode, you get a nice error page pointing you to the missing migration.  But login happens via an call to ```/login_ajax```.  If that call fails with a 500 error, it's not obvious from the UI, and you just end up with a redirect.

This can happen in the Stanford environment with the devstack because we have some schema differences that are only hit during login.  The specific problematic migration is ```common/djangoapps/student/migrations/0029_auto__add_field_userprofile_nonregistered.py```.  To fix that in our environment, since there are other migrations with number 29, use this trick

    ./manage.py lms migrate --settings=devstack --merge


### Studio

#### The View Live button gives a 404 not found error

`rake lms`, or `rake lms[dev]` will start up lms using the courseware defined in xml files in the data dir. In order for the lms to use the mongo backed DB for courseware (which is where Studio creates courses), you should use: `rake lms[cms.dev]` This will use the settings file located at lms/envs/cms/dev.py

#### A course I created in Studio isn't listed on the LMS

Be sure to run the lms as `rake lms[cms.dev]` (and you can specify IP/port numbers as listed below). This configuration cms.dev will read the courseware from the MongoDB that Studio uses. As a bit of historical information, older courseware used to run as XML from the filesystem and the existing `rake lms` uses that configuration.

### Discussion Service

* If you get a 401 error in LMS when trying to create a post, it's most likely because you haven't seeded permissions and roles for the course.

* If you added users in LMS/CMS while the discussion service was not running, you need to one-way sync your users over to the discussion service back end DB. Do this with:
```
rake django-admin[sync_user_info]
```

* Mongo db info for troubleshooting from the courseware side, for course number 999:
```
use xmodule
db.modulestore.find({'_id.course': '999', '_id.category': 'discussion'})
```
* This is how the discussion categories show up in the courseware db:
```
{ "_id" : { "tag" : "i4x", "org" : "MITx", "course" : "999", "category" : "discussion", "name" : "6a40edaeb548403caa2d24bcf827870d", "revision" : null }, "definition" : { "children" : [ ], "data" : "<discussion />\n" }, "metadata" : { "display_name" : "Discussion Tag", "discussion_category" : "Overview", "discussion_target" : "Alpha", "published_by" : 1, "discussion_id" : "c8d316ca8fc94e71ba6c8cc705e156e1", "published_date" : [ 2013, 4, 3, 20, 17, 22, 2, 93, -1 ] } }
```
* Note, if you want to remove an entire course, you can do it like this (for all courses with number 999):
```
use xmodule
db.modulestore.remove({'_id.course': '999'})
```
* Mongo db info for troubleshooting from the comment service side:
```
use cs_comments_service_development
db.contents.find()
```
* This is how comments show up in the comment service db:
```
{ "_id" : ObjectId("515c8eedb02379878b000006"), "votes" : { "up" : [ ], "down" : [ ], "up_count" : 0, "down_count" : 0, "count" : 0, "point" : 0 }, "tags_array" : [ ], "comment_count" : 0, "at_position_list" : [ ], "title" : "this should show up in alpha", "body" : "yes", "course_id" : "MITx/999/Robot_Super_Course", "commentable_id" : "c8d316ca8fc94e71ba6c8cc705e156e1", "_type" : "CommentThread", "anonymous" : false, "anonymous_to_peers" : false, "closed" : false, "author_id" : "1", "updated_at" : ISODate("2013-04-03T20:19:57.056Z"), "created_at" : ISODate("2013-04-03T20:19:57.056Z"), "last_activity_at" : ISODate("2013-04-03T20:19:57.056Z") }
```
* Discussion Forums Data formats can be found here: http://data.edx.org/internal_data_formats/discussion_data.html
* In your dev environment, to connect to a comment service other than localhost, put this into your settings.py file:
```
COMMENTS_SERVICE_URL = 'http://other_host:4567'
```