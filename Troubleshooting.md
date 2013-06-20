### Studio

* **Q:** I have created a course in Studio on my development server but when I press the View Live button to view the course in lms, I get a 404 not found error. **A:** `rake lms`, or `rake lms[dev]` will start up lms using the courseware defined in xml files in the data dir. In order for the lms to use the mongo backed DB for courseware (which is where Studio creates courses), you should use: `rake lms[cms.dev]` This will use the settings file located at lms/envs/cms/dev.py

### Discussion Service

* If you get a 401 error in LMS when trying to create a post, it's most likely because you haven't seeded permissions and roles for the course.

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
