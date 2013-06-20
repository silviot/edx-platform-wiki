## Studio

### I have created a course in Studio on my development server but when I press the View Live button to view the course in lms, I get a 404 not found error.

`rake lms`, or `rake lms[dev]` will start up lms using the courseware defined in xml files in the data dir.

In order for the lms to use the mongo backed DB for courseware (which is where Studio creates courses), you should use:
`rake lms[cms.dev]`
This will use the settings file located at lms/envs/cms/dev.py

## Discussion Service

### Handy commands to know

#### Comment Service
* To run the automated tests:
`rspec spec`
* To delete the comment service db:
`mongo cs_comments_service_development --eval "db.dropDatabase()"`
* To reinitialize the comment service db:
`bundle exec rake db:init`
* To re-index the db (must have some data in it):
`bundle exec rake db:reindex_search`
* To start up the comment service:
`ruby app.rb`
* In your dev environment, to connect to a comment service other than localhost, put this into your settings.py file:
`COMMENTS_SERVICE_URL = 'http://other_host:4567'`