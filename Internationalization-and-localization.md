We are currently hosting all translations of Open edX framework on www.transifex.com. Please feel free to make translation contributions there.

You should also join the [openedx-translation](https://groups.google.com/forum/#!forum/openedx-translation) Google Group.

In order to run your Open edX instance under a different spoken language, for instance for [Spanish (Latin American)](https://www.transifex.com/projects/p/edx-platform/language/es_419/):

1. Switch to edxapp environment:

        sudo -H -u edxapp bash
        source /edx/app/edxapp/edxapp_env
        cd /edx/app/edxapp/edx-platform

2. Configure your `~/.transifexrc` file:

        [https://www.transifex.com]
        hostname = https://www.transifex.com
        username = user
        password = pass
        token =        
        
2. Configure your `~/.transifexrc` file:

        [https://www.transifex.com]
        hostname = https://www.transifex.com
        username = user
        password = pass
        token =

    Token is left blank. You have to have permissions for the project (edx-platform) AFAIK - https://www.transifex.com/projects/p/edx-platform/ 

3. All of the languages on Transifex are already configured in the edx-platform repo.  If you've added a new language to Transifex, and we haven't added it to the configuration yet, you can add it to `conf/locale/config.yaml`.

4. Configure `LANGUAGE_CODE` in your `lms/envs/common.py`. Or, for development purposes, create a dev file called `dev_LANGCODE.py` - eg `dev_es.py` - with the following: 

        from .dev import *
        
        USE_I18N = True
        LANGUAGES = ( ('es-419', 'Spanish'), )
        TIME_ZONE = 'America/Guayaquil'
        LANGUAGE_CODE = 'es-419'

  Languages need to be specified with codes Django likes, so a code that is specific on Transifex such as `"de_DE"` must be specified as `"de-de"` in these configuration files. See https://groups.google.com/forum/#!topic/openedx-translation/vrOpMKzA0kU

5. Execute the following command in your edx-platform directory with your edx-platform virtualenv. 

        $ rake i18n:robot:pull

  Note that this command will pull *reviewed* translations for **all** languages that are listed in `conf/locale/config.yaml`. To only pull down some languages, edit `conf/locale/config.yaml` appropriately.

  To pull *unreviewed* translations along with reviewed translations, edit `/edx/app/edxapp/venvs/edxapp/src/i18n-tools/i18n/transifex.py`. Particularly, the line `execute('tx pull --mode=reviewed --all')` should be changed to `execute('tx pull --all')`

6. When you launch your LMS instance you can launch it normally and things should display properly. However, if in Step 3 you created a special "dev_LANGUAGECODE" file, you'll need to launch the LMS with the environment file explicitly stated:

        $ rake lms[dev_es,0.0.0.0:8000]

7. If you experience issues:
   - Be sure your browser is set to prefer the language set in `LANGUAGE_CODE`
   - In `common/djangoapps/student/views.py`, the user's language code is tried to be obtained from a saved preferences file. So if you are having issues seeing your language served up, it may be because your User object has a different language saved as a preference. Try creating a new user in your environment, this should clear up the issue.