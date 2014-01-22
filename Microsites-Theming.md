# Warning:
This is a draft document for a newly implemented feature in the edX platform, which still has a couple outstanding issues.

Please see PR #2061 for the latest comments and status
***
## Overview

This feature enables separate, deployable "micro-themes" (e.g. branding elements, Mako template overrides, etc.) for subdomains (e.g. foo.edx.org, bar.edx.org) in order to provide branding "multi-tenancy".

## Django configuration settings used specifically for microsites

* FEATURES('USE_MICROSITES') - Boolean toggle for enabling the microsites middleware.
* MICROSITE_NAMES - list of microsites
* MICROSITE_ROOT_DIR - directory containing subdirectories of assets for each microsite including css, images, template overrides
* MICROSITE_CONFIGURATION - A dict of dicts. Each key in the microsite configuration dict specifies the config for a microsite. There should be a configuration for each of the items in the MICROSITE_NAMES list. Keys within each microsite's config are as follows:
 * domain_prefix 
 * university 
 * platform_name 
 * logo_image_url 
 * email_from_address 
 * payment_support_email 
 * ENABLE_MKTG_SITE 
 * SITE_NAME 
 * course_org_filter 
 * course_about_show_social_links 
 * css_overrides_file 
 * show_partners 
 * show_homepage_promo_video 
 * course_index_overlay_text 
 * course_index_overlay_logo_file 
 * homepage_overlay_html 

## Other Django configuration settings that are important to microsites configuration

* VIRTUAL_UNIVERSITIES - list of university landing pages to display on the lms home page. These will be displayed whether or not they have courses created for that org. Clicking on the link for a given university will bring the user to the course catalog for the university.
* FEATURES('ENABLE_MKTG_SITE') - This is used to determine whether or not certain links (e.g. course listing page, FAQ, etc.) should be handled by the lms django app or by an external app. See [[Alternate site for marketing links]]
* MAKO_TEMPLATES['main'] - list of directories to look in for the compiled template files. The microsite root directory gets appended to this list.
* SUBDOMAIN_BRANDING - Boolean toggle for signaling that the microsites configuration dict has contents???

## Template Changes