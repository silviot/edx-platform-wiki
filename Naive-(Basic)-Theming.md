This page will detail how to do theming at the most basic level: what you need to do to specify your company's logos, site name, and (optionally) custom name for Studio.

## Specifying Site Name

In the LMS system, you need to set your PLATFORM_NAME in the environment file. Studio will import this value from the LMS.

https://github.com/edx/edx-platform/blob/master/lms/envs/common.py#L47

## Customizing Studio Name

In the CMS system, you need to set the STUDIO_NAME in the environment file. 

Optionally, you can set a STUDIO_SHORT_NAME, as well. You might do this if the formal name for your Studio system is long, such as "Hyrule University Content Management System", but colloquially everyone calls it "Hyrule CMS". We use the STUDIO_NAME only for titles; we use the STUDIO_SHORT_NAME when referring to the product within sentences.

https://github.com/edx/edx-platform/blob/master/cms/envs/common.py#L50


## Specifying Logos

To specify your site logos, override the default logo files in these directories:

### LMS
https://github.com/edx/edx-platform/tree/master/lms/static/images/default-theme

### CMS
https://github.com/edx/edx-platform/tree/master/cms/static/images/default-theme
