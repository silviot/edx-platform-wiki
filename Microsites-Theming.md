
## Overview

This feature enables separate, deployable "micro-themes" (e.g. branding elements, Mako template overrides, etc.) for subdomains (e.g. foo.edx.org, bar.edx.org) in order to provide "branding multi-tenancy". This feature also supports the ability (as a configurable option, see below) to map a subdomain to an "ORG" in the courseware catalog. For example, if you wanted foo.edx.org to only show the courses with an "ORG"="Foo" (or "FooX"). Likewise, with this filtering, the User Dashboard on foo.edx.org would only show registered courses in the same ORG filtering. Conversely, in this case, "ORG"="Foo" courses would be filtered *out* of all other course catalog listing on other subdomain, e.g. bar.edx.org would not show "ORG"="Foo" courses.

Also, this catalog partitioning also applies on the user's Dashboard. For example, if a user is enrolled in many courses across multiple Microsites, he/she will only see the courses associated with the Microsite on that Microsite's Dashboard.

This work is similar to the 'Stanford' theming that the folks at Stanford university contributed to the platform in Summer, 2013. However, the 'Stanford' theming is strictly single-tenant and doesn't support this course catalog partitioning across subdomains.

There are some limitations to Microsites compared to the 'Stanford' theming:

- One cannot use Sass or any compiled means to generate .css at 'deploy time'. However, one could use Sass at design time and compile to .css. At this point in time, our current deployment scripts at edX do Sass compilation at deploy time.

IMPORTANT NOTE: While this does allow for multi-tenancy with respect to visual branding, there is no multi-tenancy at the User database level. Therefore it is the same user account on foo.mydomain.com and bar.mydomain.com, i.e. if I had an account on www.mydomain.com, it'd be the same account on foo.mydomain.com as well as bar.mydomain.com. We'd like to extend the ability to have separate user accounts per subdomain, this work is still TBD. If you have interest in taking this on as an Open Source contribution, let us know. There are a few things to consider, such as also partitioning Sessions between subdomains (so a active login on foo.mydomain.com isn't picked up on bar.mydomain.com), etc.

## How to define a 'Microsite'

There are two main things that you will have to do to use Microsites:

* Define a Django configuration (see below)

* Provide a directory (typically a peer directory to edx-platform) which contains all of the microsite content and Mako template overrides.

## Django configuration settings used specifically for microsites.

These settings will appear - in devstack and fullstack development environments - in the lms.env.json file that can be found on /edx/app/edxapp. 

* **FEATURES['USE_MICROSITES']** - This must be set to true to turn on the Microsites feature.
* **MICROSITE_ROOT_DIR** - a directory that contains subdirectories of assets for each microsite including css, image and template overrides. The assets for each microsite are located under a subdirectory with the same name as the microsite key in the MICROSITE_CONFIGURATION dict.
* **MICROSITE_CONFIGURATION** - A dict of dicts. The existence of this dict in effect enables the microsites feature. Each key in the microsite configuration dict specifies the config for that microsite. The key name *must* match the directory inside the MICROSITE_ROOT_DIR otherwise the microsite definition will not be valid. The data within each microsites config are as follows:
 * **cybersource_config_key** - If the Microsite needs to use a different CyberSource account than the default configuration, then this should be set to the key name of the configuration section that is in CC_PROCESSOR. Addition details on using distinct CyberSource accounts can be found below.
 * **domain_prefix** - the subdomain that specifies the microsite. E.g. for foo.mydomain.com it would be "foo"
 * **university** - the university associated with the microsite. E.g. "FooX". This is a legacy concept in the Open edX code base, but it should be defined. It is recommended to set it to be the same as **domain_prefix***
 * **platform_name** - Optional path to override the value of PLATFORM_NAME. Used by many templates in the edx_platform applications. For example if you wanted the name of your site to be "My Awesome University", then this is where to set it.
 * **favicon_path** - Optional path to override the value of FAVICON_PATH.
 * **css_overrides_file** - path to stylesheet for css overrides. This is relative to the MICROSITE_ROOT_DIR. See below for more information on how to define CSS overrides.
 * **logo_image_url** - path to the branded logo image to use. This is relative to the MICROSITE_ROOT_DIR
 * **google_analytics_file** - optional override for '../google_analytics.html' which is included in the header of all application pages
 * **email_from_address** - From address for emails. Used by account creation, paid cert order, direct enrollment via instructor dashboard, etc. Overrides DEFAULT_FROM_EMAIL.
 * **payment_support_email** - Payment support email address, used for the email sent when a payment processing error occurs. Overrides PAYMENT_SUPPORT_EMAIL.
 * **ENABLE_MKTG_SITE** - Optional boolean toggle to override ENABLE_MKTG_SITE for this microsite. This should always be set to False.
 * **SITE_NAME** - hostname portion of the URI for the microsite web address. E.g. "foo.mydomain.com"
 * **course_org_filter** - string value to use for displaying the correct courses on the student dashboard. For example, if all courses are being created in Studio with ORG="MyDomainX", and if you just want to show just MyDomainX courses on your Microsite, then use the value.
 * **show_partners** - boolean toggle for displaying the edX university partners (MIT, Harvard, etc.) on the microsite's home page. This should always be set to False.
 * **homepage_overlay_html** - Optional html that is shown on the microsite's home page in place of the "\<h1>The Future of Online Education\</h1>\<h2>For anyone, anywhere, anytime\</h2>" tag line.
 * **show_homepage_promo_video** - boolean toggle for displaying a video on the microsite's home page. Defaults to True.
 * **homepage_promo_video_youtube_id** - Override for the video modal that is displayed on the microsite's home page. Defaults to "XNaiOGxWeto".
 * **course_index_overlay_logo_file** - Optional override for the logo file on the course index page. Defaults to "images/edx_bw.png"
 * **course_index_overlay_text** - Text to override the "Explore free courses from leading universities." tag line that is shown on the course index page.
 * **course_about_show_social_links** - boolean toggle for displaying the social links in the course about page's sidebar. Defaults to True.
 * **course_about_twitter_account** - Optional override for the account on the twitter social link. Defaults to "@edxonline".
 * **course_about_facebook_link** - Optional override for URL of the facebook social link. Defaults to "http://www.facebook.com/EdxOnline".
 * **COURSE_CATALOG_VISIBILITY_PERMISSION** - Optional override for the permissions check regarding allowing visibility of a course to appear on the homepage and courses listing. Suggested setting should be "see_in_catalog".
 * **COURSE_ABOUT_VISIBILITY_PERMISSION** - Optional override for the permissions check regarding allowing visibility of the Course About page. Suggested setting should be "see_about_page"
 * **ENABLE_PAID_COURSE_REGISTRATION** - Optional override for the global settings.FEATURE["ENABLE_PAID_COURSE_REGISTRATION"]. If the Microsite needs to support paywalled courses in an deployment environment which has this global setting to False, then the Microsite should set it to True.
 * **ENABLE_SHOPPING_CART** - Optional override for the global settings.FEATURE["ENABLE_SHOPPING_CART"]. If the Microsite needs to support the Shopping Cart button in a deployment environment which has this global setting to False, then the Microsite should set it to True.
 * **ENABLE_THIRD_PARTY_AUTH** - Optional override for the global settings.FEATURE["ENABLE_THIRD_PARTY_AUTH"]. If the Microsite cannot support third party authentication in an deployed environment in which the global setting is True, then the Microsite should set it to False.
 * **ALLOW_AUTOMATED_SIGNUPS** - Optional override for the global settings.FEATURE["ALLOW_AUTOMATED_SIGNUPS"] to show/hide the Auto-Register and Enroll feature in the Instructor dashboard.
 * **ALWAYS_REDIRECT_HOMEPAGE_TO_DASHBOARD_FOR_AUTHENTICATED_USER** - Setting this to False will stop the default behavior of redirecting logged in users when the land on the homepage from redirecting to the dashboard. Recommended setting is False.
 * **course_email_template_name** - Name of the CourseEmailTemplate to use when sending course emails. If the Microsite site needs to send a branded Course Email, use the Django Admin website to define an appropriate Course Email Template. Then set the configuration to point to this Template name that was used int the admin pages.
 * **course_email_from_addr** - The "from" address to use when sending course emails.
* **SUBDOMAIN_BRANDING** - dict that maps a subdomain to a university. e.g. {'foo' : 'FooX'}. NOTE: This is a legacy construct and you will not need to set this yourself.

## Other Django configuration settings that are important to microsites configuration

Note, this is informational. You should not need to manipulate these settings as they get managed in the enable_microsites() method.

* **FEATURES('ENABLE_MKTG_SITE')** - This is used to determine whether or not certain links (e.g. course listing page, FAQ, etc.) should be handled by the lms django app or by an external app. See [[Alternate site for marketing links]]
* **MAKO_TEMPLATES['main']** - list of directories to look in for the compiled template files. The microsites' root directories are appended to this list.
* **VIRTUAL_UNIVERSITIES** - list of university landing pages to display on the lms home page. These will be displayed whether or not they have courses created for the universities. Clicking on the link for a given university will bring the user to the course catalog for that university.
* **FEATURES('USE_MICROSITES')** - Internal Boolean toggle for enabling the microsites feature. This is not configured directly in a settings.py file, but instead is overridden in the code itself.


## Mako Template and Styling overrides

Create a directory with the same name as the microsite key under MICROSITE_ROOT_DIR. Under this directory create 3 folders. See common/test/test_microsites/test_microsite for examples.
* **css** - create a .css file under this dir and specify it with the css_overrides_file setting
* **images** - create this folder, and in it put your background image, header logo, etc. Specify them with the logo_image_url and course_index_overlay_logo_file settings.
* **templates** - create a directory named "templates" (hard coded. See common/djangoapps/microsite_configuration/middleware.py method get_microsite_template_path). Under here you would put any mako templates you want to override. These must have the same relative paths as the templates under /lms/templates. For example, if you wanted to override the emails/activation_email.txt, then it should have the same path in your overrides directory (e.g. emails/activation_email.text).

When changing any of the assets inside of the MICROSITE_ROOT_DIR, 'compile_assets' must be run in order to compile them into the Django asset pipeline. To do on a devstack or a production stack, use the instructions found in the operations management Wiki: https://github.com/edx/configuration/wiki/edX-Managing-the-Production-Stack#compile-assets-manually


## Examples

TBD