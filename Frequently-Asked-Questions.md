#### How can I get a translation of the Open edX platform into a different spoken language

We are using Transifex as our translation platform. We have a number of translations in progress now. Please go to our Transifex page at: https://www.transifex.com/projects/p/edx-platform/

#### I haven't received the email to activate my newly created account

When running with the development environment settings, registration emails are not sent out. However, the URL with their activation key is logged to the console. Just copy it from there and paste it into the browser.

To make the registration emails being sent out, change the file edx-platform/cms/envs/common.py the line

EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'

to the line

EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'

#### How do I make my account global staff?

When you make a new account on your local dev environment, it will be a standard student account. You can drop into the sql shell and make your account global staff, which means that it will automatically have access to "Instructor" tab on all courses in the LMS. To do this, drop into the sql shell:

`./manage.py lms dbshell`

The following command will make all local users global staff:

`update auth_user set is_staff=1;`

A similar command will make all local users global superusers, which means they'll have unrestricted access to `localhost:8000/admin`:

`update auth_user set is_superuser=1;`

#### How do I specify the image for my course?

Using Studio, upload an image named images_course_image.jpg

#### How do I restrict who can create new courses?

Please read: https://github.com/edx/edx-platform/wiki/Controlling-course-creation-rights

#### How to make the site accessible to external devices?

Modify the server deployment commands to include the IP address, like this: `rake cms[dev,0.0.0.0:8001]` and `rake lms[cms.dev,0.0.0.0:8000]`.

From a [stackoverflow question on 0.0.0.0 in Django](http://stackoverflow.com/questions/1621457/about-ip-0-0-0-0django),

> 0.0.0.0:80 is a shortcut meaning "bind to all IP addresses this computer supports". 127.0.0.1:80 makes it bind only to the "lo" or "loopback" interface. If you have just one NIC with just one IP address, you could bind to it explicitly with, say, "192.168.1.1:80" (if 192.168.1.1 was your IP address), or you could list all the IPs your computer responds to, but 0.0.0.0:80 is a shortcut for that.

### Database

#### Why use three different databases (MySQL, SQLite & MongoDB) rather than a single one?

'Courseware content' is stored in MongoDB and user-data (e.g. user tables, user state, authN, authZ, etc.) are stored in a relational database. For localdev environments, SQLite is used for the relational database to keep dependencies simple, for production environments mysql is used as the relational database. So there's actually really just two rather than three, from that perspective.

In the history of edX, the use of MongoDB as a contentstore is relatively new (previous architectures used XML-based files from the filesystem) and it seemed like a good match since courseware are well modeled as flexible JSON documents. The use of relational datastores for user-data predated the transition of the contentstores and we did not migrate those databases to a NoSQL platform.

#### Are the CMS and LMS suppose to use the same database or separate ones when in production?

CMS and LMS should share the same databases (mysql/RDS for user-data and MongoDB for courseware content and discussion forums) for any deployments. SSO is accomplished by both Django Apps using shared cookies (e.g.   *.edx.org, in our case) as well as the same Session database. So if you're deployment is on *.foo.com, then the cookie domain should be set to .foo.com, where the LMS is on www.foo.com and the CMS is on studio.foo.com.

#### With django-admin syncdb and migrate - is that run both in LMS and CMS or is it sufficient to run once?

Yes, there are no new CMS specific tables in Django, so you should just have to run it once.

#### Why doesn't the preview button work in Studio

You need to put the server name in the PREVIEW_LMS_BASE variable, found in the cms.env.json and lmd.env.json files.