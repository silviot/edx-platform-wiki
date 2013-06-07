## Databases

### Why use three different databases (MySQL, SQLite & MongoDB) rather than a single one?

'Courseware content' is stored in MongoDB and user-data (e.g. user tables, user state, authN, authZ, etc.) are stored in a relational database. For localdev environments, SQLite is used for the relational database to keep dependencies simple, for production environments mysql is used as the relational database. So there's actually really just two rather than three, from that perspective.

Historically speaking, using MongoDB as a contentstore is relatively new (previous architectures used XML-based files from the filesystem) and it seemed like a good match since courseware are well modeled as flexible JSON documents. The use of relational datastores for user-data predated the transition of the contentstores and we did not migrate those databases to a NoSQL platform.

### Are the CMS and LMS suppose to use the same database or separate ones when in production?

CMS and LMS should share the same databases (mysql/RDS for user-data and MongoDB for courseware content) for any deployments. SSO is accomplished by both Django Apps using shared cookies (e.g.   *.edx.org, in our case) as well as the same Session database. So if you're deployment is on *.foo.com, then the cookie domain should be set to .foo.com, where the LMS is on www.foo.com and the CMS is on studio.foo.com.

### With django-admin syncdb and migrate - is that run both in LMS and CMS or is it sufficient to run once?

Yes, there are no new CMS specific tables in Django, so you should just have to run it once.
