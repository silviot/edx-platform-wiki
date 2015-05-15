We are currently hosting all translations of Open edX framework on www.transifex.com. Please feel free to make translation contributions there.

You should also join the [openedx-translation](https://groups.google.com/forum/#!forum/openedx-translation) Google Group.

In order to run your Open edX instance under a different spoken language, for instance for [Spanish (Latin American)](https://www.transifex.com/projects/p/edx-platform/language/es_419/):

1. Configure your `~/.transifexrc` file:

        [https://www.transifex.com]
        hostname = https://www.transifex.com
        username = user
        password = pass
        token =

    Token is left blank. You have to have permissions for the project (edx-platform) AFAIK - https://www.transifex.com/projects/p/edx-platform/ (it is free to sign up and join this project as a translator).

     For help with Transifex configuration, see the Transifex documentation: http://docs.transifex.com/developer/client/setup#configuration

2. Switch to edxapp environment:

        sudo -H -u edxapp bash
        source /edx/app/edxapp/edxapp_env
        cd /edx/app/edxapp/edx-platform
  
        


3. Make sure all languages you wish to download are present and uncommented in `conf/locale/config.yaml`. For example, if you wish to download Arabic and Chinese (China), make sure your config.yaml file looks like this: 

        locales:
            - ar  # Arabic
            - zh_CN  # Chinese (China)


4. Configure `LANGUAGE_CODE` in your `lms/envs/common.py`. Or, for development purposes, create a dev file called `dev_LANGCODE.py` - eg `dev_es.py` - with the following: 

        from .dev import *
        
        USE_I18N = True
        LANGUAGES = ( ('es-419', 'Spanish'), )
        TIME_ZONE = 'America/Guayaquil'
        LANGUAGE_CODE = 'es-419'

  Languages need to be specified with codes Django likes, so a code that is specific on Transifex such as `"de_DE"` must be specified as `"de-de"` in these configuration files. See https://groups.google.com/forum/#!topic/openedx-translation/vrOpMKzA0kU

5. Execute the following command in your edx-platform directory with your edx-platform virtualenv. 

        $ paver i18n_robot_pull

  Note that this command will pull *reviewed* translations for **all** languages that are listed in `conf/locale/config.yaml`. To only pull down some languages, edit `conf/locale/config.yaml` appropriately.

  To pull *unreviewed* translations along with reviewed translations, edit `/edx/app/edxapp/venvs/edxapp/src/i18n-tools/i18n/transifex.py`. Particularly, the line `execute('tx pull --mode=reviewed --all')` should be changed to `execute('tx pull --all')`

6. When you launch your LMS instance you can launch it normally and things should display properly. However, if in Step 3 you created a special "dev_LANGUAGECODE" file, you'll need to launch the LMS with the environment file explicitly stated:

        $ paver lms -s dev_es -p 8000

7. If you experience issues:
   - Be sure your browser is set to prefer the language set in `LANGUAGE_CODE`
   - In `common/djangoapps/student/views.py`, the user's language code is tried to be obtained from a saved preferences file. So if you are having issues seeing your language served up, it may be because your User object has a different language saved as a preference. Try creating a new user in your environment, this should clear up the issue.

# Releasing A Language

Setting `LANGUAGE_CODE` sets one language to be your installation's default language. What if you want to support more than one language?

To "release" a second (or third, or hundredth) language, you have to configure the languages on the admin panel:

YourAwesomeDomain.com/admin/dark_lang/darklangconfig/

The `LANGUAGE_CODE` variable is for your server's default language. And then to "release" a language, you have to turn them on in the dark lang config on the admin panel - do that by going to the admin url above, then adding language codes for all additional languages you wish to release in a comma separated list. For example, to release French and Chinese (China), you'd add: 

        fr, zh-cn

to the dark lang config list. You don't need to add the language code for your server's default language, but it's no problem if you do.

Confusing, I know. The benefit of this is that you can preview languages before you release them by appending: 

        ?lang-code=xx

to the end of any url, and `?clear-lang` to undo it. Example: 

        127.0.0.1:8000/dashboard?preview-lang=ar      # Shows your site in Arabic (if Arabic is unreleased in your instance)

        127.0.0.1:8000/dashboard?clear-lang           # Resets your session

Remember that language codes with underscores and capital letters need to be converted to using dashes and lower case letters on the edX platform. For example the language code of Chinese (Taiwan) is "zh_TW" on Transifex, but "zh-tw" on the edX system.

# Localization Process For A Local Full Stack Installation

If you have installed the [Full Stack](https://github.com/edx/configuration/wiki/edX-Full-Stack) (rather than the [Developer Stack](https://github.com/edx/configuration/wiki/edX-Developer-Stack)) to provide online courses to your institution, its usually not necessary to pull the translations of all languages from Transifex (in many cases translations of only one language is needed), while the pulled translation may need to be modified to meet your specific requirements. You can follow the steps here to do so:

1. Follow the steps 1 & 2 listed on top of this document to create a `~/.transifexrc` and switch to edxapp environment.

2. If you only need to pull translations of one language, just execute the following command in your edx-platform directory with your edx-platform virtualenv:

        $ tx pull -l <lang_code>

   The `<lang_code>` here should be replaced by the language code on Transifex of the language you want to pull (for example, `zh_CN` for `Chinese (China)`).

   Note that this step will overwrite the .po files located in `conf/locale/<lang_code>/LC_MESSAGES` with the contents retrieved from Transifex, so please only do this step when you just have a new installation, or when you really want a new version of translation from Transifex.

3. If you've your own changes of edx-platform's source code in your local installation, or the source files on Transifex is not up-to-date, you may need to extract strings manually. Just execute the following command in your edx-platform directory with your edx-platform virtualenv:

        $ paver i18n_extract

   This command will extract translatable strings into several .po files located in the `conf/locale/en/LC_MESSAGES`. After the extraction process, you can merge the newly extracted strings into the corresponding .po files (except `django.po` and `djangojs.po`, which are generated by i18n tools automatically from other .po files) located in `conf/locale/<lang_code>/LC_MESSAGES`.

4. Edit the contents of .po files located in `conf/locale/<lang_code>/LC_MESSAGES` as you wish. When you finished your modification process, re-compile the translation messages manually by executing the following command in your edx-platform directory with your edx-platform virtualenv:

        $ paver i18n_fastgenerate

   Note that there is another command called `paver i18n_generate`. The difference of `paver i18n_fastgenerate` and `paver i18n_generate` is that `paver i18n_generate` will extract strings first by using `paver i18n_extract` and then compile them, while `paver i18n_fastgenerate` just compiles them without extracting.

5. Restart your Full Stack installation according to the commands listed in the page [edX Managing the Full Stack](https://github.com/edx/configuration/wiki/edX-Managing-the-Full-Stack). Note that if you want to change the default language code of a Full Stack, you can modify the value of `LANGUAGE_CODE` inside `lms.env.json` and/or the `cms.env.json` located in `/edx/app/edxapp` before starting your Full Stack.

6. You will see updated translations after the above steps. If not, you can also try to restart the nginx server and/or clear your browser's cache.