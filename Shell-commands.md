#### Django apps (Studio and LMS)
* To get rid of existing courses. Note this will not clear out data on the django side like course groups:
```
mongo xmodule --eval "db.dropDatabase()"
mongo xcontent --eval "db.dropDatabase()"
```
* To delete one course first locate your Organization, Course Number, and Course Name in the CMS> Settings> Schedule & Details page then issues this command.  For <a href="/edX/configuration/wiki" target="_blank">configuration</a> issues with "Installing edX on a single Ubuntu 12.04 (Precise) server", please change 'env' to 'aws'.
```
#Command: rake django-admin[delete_course,cms,env,"{Organization}/{Course Number}/{Course Name} commit"]
rake django-admin[delete_course,cms,aws,"MITx/999/Robot_Super_Course commit"]
```
* To seed the permissions for the comment service for user 'robot' course MITx/999/Robot_Super_Course
```
export DJANGO_SETTINGS_MODULE=lms.envs.dev
export PYTHONPATH=.
django-admin.py seed_permissions_roles "MITx/999/Robot_Super_Course"
django-admin.py assign_role robot Moderator "MITx/999/Robot_Super_Course"
django-admin.py assign_role robot Administrator "MITx/999/Robot_Super_Course"
```
* To sync the user info from cms/lms over to the comment service:
```
rake django-admin[sync_user_info,lms,dev]
```

#### Mongo (courseware)
* To list all the courses:
```
use xmodule
db.modulestore.find( { "_id.category" : "course" }, {'name':'1'} )
```
* To find all the updates for courses numbered 999:
```
db.modulestore.find({'_id.course': '999','_id.name':'updates'})
```
* To remove courses numbered 999 from all orgs. Note this will not clear out data on the django side like course groups:
```
db.modulestore.remove({'_id.course': '999'})
```

#### Comment Service
* To run the automated tests:
```
rspec spec
```
* To delete the comment service db:
```
mongo cs_comments_service_development --eval "db.dropDatabase()"
```
* To reinitialize the comment service db:
```
bundle exec rake db:init
```
* To re-index the db (must have some data in it):
```
bundle exec rake db:reindex_search
```
* To start up the comment service:
```
ruby app.rb
```