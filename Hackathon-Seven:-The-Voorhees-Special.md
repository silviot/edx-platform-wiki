Let's focus on APIs if we can, but completely understand if we want to focus on other things.

# The Hacks

Demo day is Friday the 13th, hopefully the demos aren't too scary! Extra points for incorporating the theme into your demos!

# The Schedule
Jun 12, 2014: *hack* (6:00pm - Dinner)
Jun 13, 2014: *demo* (3:00-4:30pm) (Google Hangout, Drinks and Snacks!)

# Dinner Attendees
* Andy A.
* Gabe 
* Steve
* Adam
* Carlos
* Will
* Jay
* Ben
* Grady
* Dennis
* Ben McMorran

# Hackathon Ideas

Team | Suggested By | Idea        | Notes |
-----|--------------|-------------|------- |
| adam, mat | Adam | Analysis of "goals" field in auth_userprofile | Why do people sign up for edX? Do student goals correlate with success? Completion rates? Courses enrolled in? |
| clintonb | DB      | Django 1.6! | It's a crazy idea. But I want to give it a shot. ([the work begins here](https://github.com/edx/xblock-sdk/pull/10)); (dave: maybe 1.7? It's almost out... cale: shoot for 1.7! Also, see https://github.com/edx/edx-platform/wiki/Moving-to-Django-1.7) |
| Christina, Frances, Mark | Christina| Improvement to Advanced Settings page in Studio https://github.com/edx/edx-platform/pull/4073| Display names, help, hide "deprecated" fields, possibly validation. Add some structure; links to docs for each https://github.com/edx/edx-platform/pull/4073|
| JZ, BenP, Christine, Frances | JZ | Create a rules-based risk calculation for PRs | I got started with this [here](https://github.com/jzoldak/gh-pr-risk) |
|Ned | Ned     | Coverage measurement of Django templates | Last time was Mako, let's try Django. |
|| Will    | Mobile study groups app  |  |
|Steve |Steve   | Group Project Assessments | https://github.com/edx/edx-ora2/tree/sanchez/group_projects Allow a group of students to collaborate on a single project, submit it for review, then grade peer groups. Yay! |
||AndyA | [[XBlock Admin Views|xblock-admin-views]] | Support global/course-scoped admin pages for xblocks (for Studio, but would love help with Instructor Dashboard integration) |
| Brian, Dennis, Gabe |Gabe | Hive Data Pipeline | Load all event data (and maybe some other sources) in to hive tables to experiment with and run adhoc queries against. |
||Jarv | Packages for open-edx installations | Install edX without having to go out to pypi, github, with a package for every role. How about `apt-get install edx`? |
|dave| dave  | Faster Grading | A few different possibilities, starting with reducing SQL queries. |
|| dave  | Cert Generation via Celery Task | 
|| dave  | Improving LMS browser load times | 
|| dave  | Improving Studio browser load times |
|| dave  | edX as an OAuth provider | i.e. SSO between edX apps |
|| dave  | Submissions app as XBlock Runtime Service | 
|| dave  | Downloads service | A facility for writing files to S3 that would be made available for XBlocks. |
|| dave  | Cheap pull-only course/item/user level notifications | 
|| dave  | Video source service | abstract away different locations/encodings |
|dkh, flowerhack| dave  | A better terminal-based viewer for profiling results | i.e. a better RunSnakeRun (something like/based on https://github.com/nedbat/memsee could be cool) (dave: python -m pstats already provides basic cmdline functionality, but I was looking to make a tree listing cumulative view and other context-specific data via something like urwid) |
|| dave  | Middleware to automatically dump profiles of views in dev | to make performance debugging easier |
|| dave  | PostgreSQL 9.4 modulestore! | |
|| dave  | User info XBlock Service | |
| Brian T. | Brian T. | Help move [Sass/Bourbon/Neat update](https://github.com/edx/edx-platform/pull/3462) | Status: **Merged**|
|| cale  | Turn modulestores into FieldDatas (to continue the XBlockification of edx-platform) | |
|| cale  | Micro-app to manage ratcheting up/down of values (coverage, pep8/pylint, etc) | Sarina: Am curious what you're thinking for this |
|| cale  | Try out new load-test options (swarm? gatling?) | |
|| cale  | Test plan generator | |
|| cale  | Micro feature flags: need to migrate auth roles? Just disable things that create new auth roles | |
|| cale  | More db-backed configuration | |
|| Marco  | Make progress on LMS Redesign Phases 1-4 see: https://edx-wiki.atlassian.net/wiki/display/LMS/LMS+Redesign+-+Project+Phasing, summary  | |
|| Sef | Gather and send basic stats from open-source deployments, collect up for KPI's.  | Reqts: Opt in.  management command to preview what would be sent, send once, or send periodically (celery beat?).  Aggregate stats only: enrollments, certificates, etc.  Server to collect stats.  What transport, email? I can't participate myself, but would be a fun feature to hack in |
|| cale | Try using http://django-compressor.readthedocs.org/en/latest/ instead of django-pipeline for our assets |
|| cale | Experiment w/ using django-storages to power our static assets (thorny issue: locked assets and one-time urls for same |
|| carlos | Provide some automatic extension points for external django apps to the LMS.| The idea is to move apps like psychometrics and licenses to their own reports, but still let people easily include them in their deployments with just changes to django settings (not to url.py)|
| BenM, Nimisha | nimisha | Update [[Studio Analytics|https://edx-wiki.atlassian.net/wiki/display/STU/Studio+Analytics]]  | We already upload data using segment.io, but some is buggy and more should be tracked.|
|BenM, Nimisha | nimisha | Touch of gamification: crossword puzzle problem xBlock | Add a little "fun" way of testing students. |
|| dave | Log releases in New Relic | https://rpm.newrelic.com/accounts/88178/applications/1588430/deployments/instructions
|| nikhil | API instead of iframes for Course about pages | https://edx-wiki.atlassian.net/browse/WEB-869
|| nikhil | Integration of platform with interactive Raspberry Pi preview & code generator |
|| nikhil | Integration of platform with chat application |
|| grady | EASE --> EASiEr | I think that sums it up.  Diving into EASE, doing a deep clean. Removing Redundancy, speeding things up.  Not doing quadruple checking for ASCII formatting.  Etc. |
| Olga | Olga | Analytics Wish List | Gather questions from teams about our courses and students |
| gprice | gprice | diff-quality 2.0 | Productizing work from last hackathon |
| dmitchell | dmitchell | heartbeat | refactor heartbeat to scale and to test service |
| sarina, feanil | feanil | refactor django-pipeline | remove coffeescript and sass compilation from paver and into django-pipeline |
| sarina | sarina | language selection menu in CMS | wire up Talbs' UX prototype of CMS language selector menu |
| Will | Will | Outsource the test infrastructure | Try out SolanoLabs as a replacement for Jenkins |
| Brian & Lou | Brian & Lou | edX in a Mobile World | What would edX designed from the ground up for mobile look like? [Prototype screens can be viewed here](http://invis.io/2FXUPJKD) |
|Justin H | Justin H | Report Generation Automation | link dependent tasks to run with one command with Luigi framework