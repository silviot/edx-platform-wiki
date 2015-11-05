#### Django apps (Studio and LMS)
* To run django admin commands use manage.py with the following syntax:

```
path/to/edx-platform/manage.py  [lms|cms]  --settings=aws <cmd>
```

For example to see all of the available lms commands on the single server vagrant image:

```
sudo -u edxapp /edx/bin/python.edxapp /edx/app/edxapp/edx-platform/manage.py lms --settings=aws help
```

* To create a superuser account on the edX vagrant image
```
sudo -u edxapp /edx/bin/python.edxapp /edx/app/edxapp/edx-platform/manage.py lms --settings=aws
createsuperuser --username <user>
```


* To get rid of existing courses. Note this will not clear out data on the django side like course groups:
```
mongo xmodule --eval "db.dropDatabase()"
mongo xcontent --eval "db.dropDatabase()"
```
* To seed the permissions for the comment service for user 'robot' course MITx/999/Robot_Super_Course
```
export DJANGO_SETTINGS_MODULE=lms.envs.dev
export PYTHONPATH=.
django-admin.py seed_permissions_roles "MITx/999/Robot_Super_Course"
django-admin.py assign_role robot Moderator "MITx/999/Robot_Super_Course"
django-admin.py assign_role robot Administrator "MITx/999/Robot_Super_Course"
```

#### Mongo (courseware)
* To list all the courses:
```
use edxapp
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