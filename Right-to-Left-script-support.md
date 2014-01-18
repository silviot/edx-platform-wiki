# Warning:
This is a draft document for a feature that has not yet been merged into the edX platform.
Please see PR #2056 for the latest comments and status
***
## Overview

This page will show you how to configure your platform to use two or more languages besides the default English language. The example uses Arabic language (ar) for demonstration purposes. You will need to check the Transifex project at https://www.transifex.com/projects/p/edx-platform/ for your desired language and just substitute in with the desired language code.


## Django configuration settings

These set of settings should be added to a separate environment file that imports the main settings file you are using. These will tell django that this project uses i18n and the languages you want to use (Please refer to django i18n documentation [here](https://docs.djangoproject.com/en/1.4/topics/i18n/translation/)). For example, on my local dev machine I created a file in both lms/envs/cms and cms/envs and named it dev_ar.py with the following settings,
```
from dev import *
from django.conf.global_settings import LANGUAGES

USE_I18N = True
TIME_ZONE = 'Asia/Amman'
LANGUAGE_CODE = 'en-us'

ugettext = lambda s: s
LANGUAGES = (
    ('ar', ugettext('Arabic')),
    ('en', ugettext('English')),
)
```

Since this is all local changes and testing, I add these settings to my dev.py files in both the lms and cms (make sure to revert these changes if you are willing to submit a PR)

You also need to go to your conf/locale/config file and add the language(s) you are going to use to the on the locales list.

`"locales" : ["en", "ar"],`

In my case I am using Arabic next to English, so the locale list will like above.

If you are using an RTL language and you need a right-to-left view of you platform, you need to tell django to compile the new -rtl.sass files that has been added to the project. This is done by **adding** the following to the PIPELINE_CSS dictionary setting in common.py in the cms/envs/ for the CMS,
```
'style-app-rtl': {
        'source_filenames': [
            'sass/style-app-rtl.css',
        ],
        'output_filename': 'css/cms-style-app-rtl.css',
    },

'style-app-extend1-rtl': {
        'source_filenames': [
            'sass/style-app-extend1-rtl.css',
        ],
        'output_filename': 'css/cms-style-app-extend1-rtl.css',
    },
'style-xmodule-rtl': {
        'source_filenames': [
            'sass/style-xmodule-rtl.css',
        ],
        'output_filename': 'css/cms-style-xmodule-rtl.css',
    },
```
and the following in common.py in lms/envs/ for the LMS,
```
'style-app-rtl': {
        'source_filenames': [
            'sass/application-rtl.css',
            'sass/ie.css'
        ],
        'output_filename': 'css/lms-style-app-rtl.css',
    },
'style-app-extend1-rtl': {
        'source_filenames': [
            'sass/application-extend1-rtl.css',
        ],
        'output_filename': 'css/lms-style-app-extend1.css',
    },
'style-app-extend2-rtl': {
        'source_filenames': [
            'sass/application-extend2-rtl.css',
        ],
        'output_filename': 'css/lms-style-app-extend2-rtl.css',
    },
'style-course-rtl': {
        'source_filenames': [
            'sass/course-rtl.css',
            'xmodule/modules.css',
        ],
        'output_filename': 'css/lms-style-course-rtl.css',
    },
```
The last step in configuring django is to add the url to the django veiw that is responsible for language switching. So at the end of urls.py in both LMS and CMS you have to add the following,

`url(r'^i18n/', include('django.conf.urls.i18n')),`


## Importing language files from Transifex

First, you need to register with transifex and get yourself a username and password. Send a request to join one or more of the language teams to be able to access the edX language files.

Install the transifex-client using `pip install transifex-client` or `easy_install transifex-client`. 

(Notes posted on openedx-translation from Andres Lucena (Thank you!), tested and currently in use)

Configure your  ~/.transifexrc file:
```
[https://www.transifex.com]
hostname = https://www.transifex.com
username = user
password = pass
token =
```
Token is left blank. You have to have permissions for the project (edx-platform) AFAIK - https://www.transifex.com/projects/p/edx-platform/ .

Execute the following command in your shell to pull the language files to your local machine,

(example of pulling the Arabic language files)
`$ tx pull -l ar`
or 
`$ tx pull -l <language_to_use>`

You should have your language .po files in conf/locale/ar (or conf/locale/<langauge_to_use>).



## Compiling language files

The edX team has put a simple one line command to do this, just type in 

`rake i18n:generate`

For this command to execute successfully with the settings I am using, you need to have the four .po language files for both English and Arabic. If you don't have them already for English, just pull the en_us files and copy them to the conf/locale/en/LC_MESSAGES directory, or change the config file to use 'en_us' instead of 'en'.



## Adding HTML to switch languages

*(Please note that the edX UI team are working on a fixed language switcher for the platform, please make sure you do not submit any of this code if you are hissing a PR)

To be able to switch between your languages, you need to add the language switcher form. A very easy to use form is already available on django documentation, all it needs is a minimal changes to work inside the mako templates. This is the form I used to switch languages first.
```
<form action="/i18n/setlang/" method="POST" style="display: none" id="language_form">
    <input type="hidden" name="csrfmiddlewaretoken" value="${ csrf_token }"/>
    <!--<input name="next" type="hidden" value="/next/page/" />-->
    <select name="language" id="language_code">
        % for lang in settings.LANGUAGES:
                <option value="${ lang[0] }">${ lang[1] }</option>
        % endfor
    </select>
    <input type="submit" value="Go" />
</form>
```
To be able to see this form, I added it the navigation.html file in lms/templates for the LMS and in header.html in cms/templates/widgets for the CMS. Make sure you don't add it inside an “if” closure so you would be able to see it at all times.


## Starting the project

Start the project with the new environment file created (if you created a new one) as so,

`rake lms[cms.dev_ar]`

or 

`rake cms[dev_ar]`

And from here you should be able to switch between your configured language using the language switch form you added to the header of your LMS and CMS. If the language is uses an RTL view, django will take care of figuring out that this is an RTL language by itself.