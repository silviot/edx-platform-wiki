## Studio

### I have created a course in Studio on my development server but when I press the View Live button to view the course in lms, I get a 404 not found error.

`rake lms`, or `rake lms[dev]` will start up lms using the courseware defined in xml files in the data dir.

In order for the lms to use the mongo backed DB for courseware (which is where Studio creates courses), you should use:
`rake lms[cms.dev]`
This will use the settings file located at lms/envs/cms/dev.py
