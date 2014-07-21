_This feature was added to the edX platform in July, 2013._

# Controlling Course Creation Rights

There are a couple of settings you can enable to allow control over who can create new courses in Studio.

NOTE: We use the phrase "is_staff" to indicate the bit in the auth_user table that is the standard Django identity framework. This bit grants users with "admin" rights to the site and the ability to see/edit all content. Readers of this document might get confused between "is_staff" - the Django setting - and our use of the words "staff" or "course staff" which are application level permissions groups to indicate users that have been authorized to create/edit content.

## Disable course creation for users not marked "is_staff"
You can completely disable the ability for any user who is not marked "is_staff" to create a new course. Users who have been added as staff to existing courses will be able to view and edit those courses, but the ability to create a new course will only be shown to users with the "is_staff" Django user setting.

### Disabling course creation
In /edx-platform/cms/envs/common.py, or your extension of it, add 'DISABLE_COURSE_CREATION': True to MITX_FEATURES.

### Marking a user as "is_staff"
There is a Django admin command for marking an account as "is_staff": 
```
./manage.py lms set_staff emailaddress
```

Note that "is_staff" is a [Django auth field](https://docs.djangoproject.com/en/dev/ref/contrib/auth/). It is completely unrelated to the course staff concept within Studio.

### Setting a contact e-mail address
If DISABLE_COURSE_CREATION is True, users not marked "is_staff" may see a message on the Studio dashboard prompting them to e-mail if they need a course to be created. This message will only be shown if an e-mail address has been set as the value of 'STUDIO_REQUEST_EMAIL' in MITX_FEATURES in /edx-platform/cms/envs/common.py (or your extension of it). You should also customize the language in this message so it does not refer to edX (/edx-platform/cms/templates/index.html)

## Selectively enable course creation
You can grant and revoke course creation rights to individual users via a Django admin table. User accounts marked "is_staff" do not need to go through this process-- those users are always allowed to create courses, and there is not a way to revoke their access.

### Enabling control of course creation rights
In /edx-platform/cms/envs/common.py, or your extension of it, set 'ENABLE_CREATOR_GROUP': True to MITX_FEATURES. Note that DISABLE_COURSE_CREATION (discussed above) takes precedence over ENABLE_CREATOR_GROUP, so make sure that DISABLE_COURSE_CREATION is not set to True.

```
### 1) Update ENABLE_CREATOR_GROUP in common.py
cd /edx/app/edxapp/edx-platform/cms/envs
sudo nano common.py

Set :
FEATURES {
    ENABLE_CREATOR_GROUP : True
}

### 2) Update the database structure of the CMS and reboot the CMS
cd /edx/app/edxapp/edx-platform
# Update CMS DB structure
sudo -u www-data /edx/bin/python.edxapp ./manage.py cms syncdb --migrate --settings aws --migrate –noinput
# Reboot
sudo /edx/bin/supervisorctl -c /edx/etc/supervisord.conf restart edxapp:
```
### User workflow
If ENABLE_CREATOR_GROUP is set to True, this is the workflow for a new Studio user:

1. User creates a new account. Their status in the admin table will be "unrequested".
1. When the user goes to the Studio dashboard, they will see a message about becoming a course creator in Studio.
1. The user can request the ability to create courses, which changes their status in the admin table to "pending".
1. The user sees on the dashboard that they are in the "pending" state. When you grant the user access, they will receive an e-mail letting them know that they should return to the Studio dashboard. At that point, they will be able to create courses.
1. If you deny the user access, they will also receive an e-mail message. When they go to the Studio dashboard, they will see a message saying that they have been denied access. 

_Caveat-- language in these messages is specific to edX and our xConsortium partners. We recommend that you modify this language for your instance of the platform (/edx-platform/cms/templates/index.html)._

### The Course Creator Admin Table
The url for the course creator admin table is /admin/course_creators/coursecreator/ (relative to where you are running your instance of the edX platform). You must log in with a username for an account marked "is_staff". See above for instructions on how to mark an account as "is_staff".

### Setting a contact e-mail address
You should set a contact e-mail address as the value of 'STUDIO_REQUEST_EMAIL' in MITX_FEATURES in /edx-platform/cms/envs/common.py (or your extension of it). This address will receive notification when someone has requested course creation access, and it will be included in e-mails sent out to users when their course creation status changes.

### Grandfathering existing users
If you wish to grant course creation access to all existing course instructors (users who have previously created courses) on your instance of the platform, you can do so via a Django admin command: ./manage.py cms populate_creators

Course staff (users who have been added to courses as staff, but did not originally create the courses) will be added to the table with status "unrequested".
