_This feature was added to the edX platform in July, 2013._

# Controlling Course Creation Rights

There are a couple of settings you can enable to allow control over who can create new courses.

## Disable course creation for users not marked "is_staff"
You can completely disable the ability for any user who is not marked "is_staff" to create a new course. Users who have been added as staff to existing courses will be able to view and edit those courses, but the ability to create a new course will only be shown to users with the "is_staff" Django user setting.

### Disabling course creation
In /edx-platform/cms/envs/common.py, or your extension of it, add _'DISABLE_COURSE_CREATION': True_ to MITX_FEATURES.

### Marking a user as "is_staff"
There is a Django admin command for marking an account as "is_staff": _/manage.py lms set_staff emailaddress_

### Setting an e-mail address for questions
If DISABLE_COURSE_CREATION is True, users (not marked "is_staff") can see a message on the Studio dashboard prompting them to e-mail if they need a course to be created. This message will only be shown if 'STUDIO_REQUEST_EMAIL' has been set in MITX_FEATURES in /edx-platform/cms/envs/common.py (or your extension of it). 
