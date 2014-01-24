# Warning:
This is a draft document for a newly implemented feature in the edX platform, which still has a couple outstanding issues.

Please see PR #2061 for the latest comments and status
***
## Overview

This feature enables separate, deployable "micro-themes" (e.g. branding elements, Mako template overrides, etc.) for subdomains (e.g. foo.edx.org, bar.edx.org) in order to provide branding "multi-tenancy".

## Django configuration settings used specifically for microsites

* **SUBDOMAIN_BRANDING** - dict that maps a subdomain to a university. e.g. {'foo' : 'FooX'}
* **MICROSITE_ROOT_DIR** - a directory that contains subdirectories of assets for each microsite including css, image and template overrides. The assets for each microsite are located under a subdirectory with the same name as the microsite key in the MICROSITE_CONFIGURATION dict.
* **MICROSITE_CONFIGURATION** - A dict of dicts. The existence of this dict in effect enables the microsites feature. Each key in the microsite configuration dict specifies the config for that microsite. Keys within each microsite's config are as follows:
 * **domain_prefix** - the subdomain that specifies the microsite. E.g. for foo.edx.org it would be "foo"
 * **university** - the university associated with the microsite. E.g. "FooX"
 * **platform_name** - Optional path to override the value of PLATFORM_NAME. Used by many templates in the edx_platform applications.
 * **favicon_path** - Optional path to override the value of FAVICON_PATH
 * **css_overrides_file** - path to stylesheet for css overrides. This is relative to the MICROSOFT_ROOT_DIR
 * **logo_image_url** - path to the branded logo image to use. This is relative to the MICROSOFT_ROOT_DIR
 * **google_analytics_file** - optional override for '../google_analytics.html' which is included in the header of all application pages
 * **email_from_address** - From address for emails. Used by account creation, paid cert order, direct enrollment via instructor dashboard, etc. Overrides DEFAULT_FROM_EMAIL.
 * **payment_support_email** - Payment support email address, used for the email sent when a payment processing error occurs. Overrides PAYMENT_SUPPORT_EMAIL.
 * **ENABLE_MKTG_SITE** - Optional boolean toggle to override ENABLE_MKTG_SITE for this microsite. Defaults to True if ENABLE_MKTG_SITE is not set at the global level.
 * **SITE_NAME** - hostname portion of the URI for the microsite web address. E.g. "foo.edx.org"
 * **course_org_filter** - string value to use for displaying the correct courses on the student dashboard
 * **show_partners** - boolean toggle for displaying the edX university partners (MIT, Harvard, etc.) on the microsite's home page. Defaults to True.
 * **homepage_overlay_html** - Optional html that is shown on the microsite's home page in place of the "\<h1>The Future of Online Education\</h1>\<h2>For anyone, anywhere, anytime\</h2>" tag line.
 * **show_homepage_promo_video** - boolean toggle for displaying a video on the microsite's home page. Defaults to True.
 * **homepage_promo_video_youtube_id** - Override for the video modal that is displayed on the microsite's home page. Defaults to "XNaiOGxWeto".
 * **course_index_overlay_logo_file** - Optional override for the logo file on the course index page. Defaults to "images/edx_bw.png"
 * **course_index_overlay_text** - Text to override the "Explore free courses from leading universities." tag line that is shown on the course index page.
 * **course_about_show_social_links** - boolean toggle for displaying the social links in the course about page's sidebar. Defaults to True.
 * **course_about_twitter_account** - Optional override for the account on the twitter social link. Defaults to "@edxonline".
 * **course_about_facebook_link** - Optional override for URL of the facebook social link. Defaults to "http://www.facebook.com/EdxOnline".


## Other Django configuration settings that are important to microsites configuration

* **FEATURES('ENABLE_MKTG_SITE')** - This is used to determine whether or not certain links (e.g. course listing page, FAQ, etc.) should be handled by the lms django app or by an external app. See [[Alternate site for marketing links]]
* **MAKO_TEMPLATES['main']** - list of directories to look in for the compiled template files. The microsites' root directories are appended to this list.
* **VIRTUAL_UNIVERSITIES** - list of university landing pages to display on the lms home page. These will be displayed whether or not they have courses created for the universities. Clicking on the link for a given university will bring the user to the course catalog for that university.
* **FEATURES('USE_MICROSITES')** - Internal Boolean toggle for enabling the microsites feature. This is not configured directly in a settings.py file, but instead is overridden in the code itself.


## Styling overrides

Create a directory with the same name as the microsite key under MICROSITE_ROOT_DIR. Under this directory create 3 folders. See common/test/test_microsites/test_microsite for examples.
* **css** - create a .css file under this dir and specify it with the css_overrides_file setting
* **images** - create this folder, and in it put your background image, header logo, etc. Specify them with the logo_image_url and course_index_overlay_logo_file settings.
* **templates** - create a directory named "templates" (hard coded. See common/djangoapps/microsite_configuration/middleware.py method get_microsite_template_path). Under here you would put any mako templates you want to override.
