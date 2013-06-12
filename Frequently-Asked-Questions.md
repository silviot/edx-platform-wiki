## User Activation Link

### I have my development server up and running and have registered a user, but I haven't gotten an email to activate my new account. How do I activate this account?

When running with the development environment settings, registration emails are not sent out. However, the URL with their activation key is logged to the console. Just copy it from there and paste it into the browser.

## Courses

### How do I specify the image for my course

Using Studio, upload an image named images_course_image.jpg


## Databases

### Why use three different databases (MySQL, SQLite & MongoDB) rather than a single one?

'Courseware content' is stored in MongoDB and user-data (e.g. user tables, user state, authN, authZ, etc.) are stored in a relational database. For localdev environments, SQLite is used for the relational database to keep dependencies simple, for production environments mysql is used as the relational database. So there's actually really just two rather than three, from that perspective.

In the history of edX, the use of MongoDB as a contentstore is relatively new (previous architectures used XML-based files from the filesystem) and it seemed like a good match since courseware are well modeled as flexible JSON documents. The use of relational datastores for user-data predated the transition of the contentstores and we did not migrate those databases to a NoSQL platform.

### Are the CMS and LMS suppose to use the same database or separate ones when in production?

CMS and LMS should share the same databases (mysql/RDS for user-data and MongoDB for courseware content) for any deployments. SSO is accomplished by both Django Apps using shared cookies (e.g.   *.edx.org, in our case) as well as the same Session database. So if you're deployment is on *.foo.com, then the cookie domain should be set to .foo.com, where the LMS is on www.foo.com and the CMS is on studio.foo.com.

### With django-admin syncdb and migrate - is that run both in LMS and CMS or is it sufficient to run once?

Yes, there are no new CMS specific tables in Django, so you should just have to run it once.

## Creating courses in Studio

### I created a new course in Studio, but when I go to the LMS, I don't see it listed?

Be sure to run the lms as `rake lms[cms.dev]` (and you can specify IP/port numbers as listed below). This configuration cms.dev will read the courseware from the MongoDB that Studio uses. As a bit of historical information, older courseware used to run as XML from the filesystem and the existing `rake lms` uses that configuration.

## Outside access

### How to make the site accessible to external devices?

Modify the server deployment commands to include the IP address, like this: `rake cms[dev,0.0.0.0:8001]` and `rake lms[cms.dev,0.0.0.0:8000]`.

From a [stackoverflow question on 0.0.0.0 in Django](http://stackoverflow.com/questions/1621457/about-ip-0-0-0-0django),

> 0.0.0.0:80 is a shortcut meaning "bind to all IP addresses this computer supports". 127.0.0.1:80 makes it bind only to the "lo" or "loopback" interface. If you have just one NIC with just one IP address, you could bind to it explicitly with, say, "192.168.1.1:80" (if 192.168.1.1 was your IP address), or you could list all the IPs your computer responds to, but 0.0.0.0:80 is a shortcut for that.