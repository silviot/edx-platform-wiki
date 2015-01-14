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
  
        


3. All of the languages on Transifex are already configured in the edx-platform repo.  If you've added a new language to Transifex, and we haven't added it to the configuration yet, you can add it to `conf/locale/config.yaml`.

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

To "release" a second (or third, or hundreth) language, what you have to do is to config the languages on the admin panel:

YourAwesomeDomain.com/admin/dark_lang/darklangconfig/

The `LANGUAGE_CODE` variable is for your server's default language. And then to "release" a language, you have to turn them on in the dark lang config on the admin panel - do that by going to the admin url above, then adding language codes for all additional languages you wish to release in a comma separated list. For example, to release French and Chinese (China), you'd add: 

        fr, zh-cn

to the dark lang config list. You don't need to add the language code for your server's default language, but it's no problem if you do.

Confusing, I know. The benefit of this is that you can preview languages before you release them by appending: 

        ?lang-code=xx

to the end of any url, and `?clear-lang` to undo it. Example: 

        127.0.0.1:8000/dashboard?preview-lang=ar      # Shows your site in Arabic (if Arabic is unreleased in your instance)

        127.0.0.1:8000/dashboard?clear-lang                # Resets your session

Remember than lang codes with underscores and capital letters need to be converted to using dashes and lower case letters on the edX platform. For example Chinese (Taiwan) is code "zh_TW" on Transifex, but "zh-tw" on the edX system.