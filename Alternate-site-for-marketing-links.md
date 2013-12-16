The edX Platform has the ability to integrate with an alternate site for serving pages that are likely to change at a different pace than the code base itself. For example: the FAQ page, terms of service, privacy policy, etc.

## Important django configuration settings:
* **FEATURES['ENABLE_MKTG_SITE']** - toggle to enable alternate urls for certain pages e.g. terms of service, faq, etc. This is for anything that has marketing_link('foo') in its template html.
* When ENABLE_MKTG_SITE is set to False: **MKTG_URL_LINK_MAP** - dict of marketing page names used in templates and the corresponding string that a reverse of which would resolve to the correct page. For example /lms/templates/footer.html defines these links, so in order for it to work the map would need to contain keys for "ABOUT", "JOBS", "PRESS", "FAQ", "CONTACT". Beware that we have a tricky piece of code in lms/urls.py that assumes that we have a template named foo.html and a corresponding url pattern for ^foo for every key FOO in the MKTG_URL_LINK_MAP, other than ROOT, COURSES, and FAQ which are handled separately.
* When ENABLE_MKTG_SITE is set to True: **MKTG_URLS** - dict of marketing page names used in templates (see above), along with the paths to add on to the root of the alternate site that they should navigate to. Note that they must contain a leading "/". You need to define them in this dict because otherwise django would not know how to resolve them. Note that in this dict you must define an item with the key "ROOT" which points to the root of the alternate site. E.g. "http://www.example.com"

## To run without an alternate marketing site
Use something like the following. This will make all the references you have added in MKTG_URL_LINK_MAP into the correct links. Test with the /register page for lms or the /signup page for Studio. Links you have not defined in the map will turn into # anchor tags for the page that you're on (effectively doing nothing). These will log a debug message to the console when you load a page that has a reference to one. Test with loading the main page of lms and looking at the Jobs or Press link.

settings.py for lms:

```python
FEATURES['ENABLE_MKTG_SITE'] = False
MKTG_URLS = {}
MKTG_URL_LINK_MAP = {
    'ABOUT': 'about_edx',
    'CONTACT': 'contact',
    'FAQ': 'help_edx',
    'COURSES': 'courses',
    'ROOT': 'root',
    'TOS': 'tos',
    'HONOR': 'honor',
    'PRIVACY': 'privacy_edx',
    'WHAT_IS_VERIFIED_CERT': 'verified-certificate',
}
```

or json snippet:

```json
"FEATURES": {
    "ENABLE_MKTG_SITE": false,
},
"MKTG_URLS": {},
"MKTG_URL_LINK_MAP": {
    "TOS": "tos",
    "ROOT": "root",
    "HONOR": "honor"
},
```

Studio - marketing_link('tos') and marketing_link('privacy') are used in templates, but there is no appropriate page to send them to in the codebase:

```python
FEATURES['ENABLE_MKTG_SITE'] = False
MKTG_URLS = {}
MKTG_URL_LINK_MAP = {}
```

## To run with an alternate marketing site
Use something like the following. This will make the links go to where you specified with the MKTG_URL_LINK_MAP.
Test with the /register page for lms or the /signup page for studio and looking at the URL for the Terms of Service link.  Links you have not defined in the map will turn into # anchor tags for the page that you're on (effectively doing nothing). These will log a debug message to the console when you load a page that has a reference to one. Test with loading the main page of lms and looking at the Jobs or Press link.
lms:

```python
FEATURES['ENABLE_MKTG_SITE'] = True
MKTG_URLS = {
    'ABOUT': '/about-us',
    'CONTACT': '/contact-us',
    'FAQ': '/student-faq',
    'COURSES': '/courses',
    'ROOT': 'https://www.edx.org',
    'TOS': '/edx-terms-service',
    'HONOR': '/terms',
    'PRIVACY': '/edx-privacy-policy',
    'WHAT_IS_VERIFIED_CERT': '/verified-certificate',
}

MKTG_URL_LINK_MAP = {}
```

or json syntax:

```json
"FEATURES": {
    "ENABLE_MKTG_SITE": true,
},
"MKTG_URLS": {
    "ROOT": "https://www.example.com",
    "TOS": "/my-terms-service",
    "PRIVACY": "/my-privacy-policy"
},
"MKTG_URL_LINK_MAP": {},
```

Studio:

```python
FEATURES['ENABLE_MKTG_SITE'] = True
MKTG_URLS = {
    'ROOT': 'https://www.example.com',
    'TOS': '/my-terms-service',
    'PRIVACY': '/my-privacy-policy',
}
MKTG_URL_LINK_MAP = {}
```
