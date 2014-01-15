We are currently hosting all translations of Open edX framework on www.transifex.com. Please feel free to make translation contributions there.

You should also join the [openedx-translation](https://groups.google.com/forum/#!forum/openedx-translation) Google Group.

In order to run your Open edX instance under a different spoken language, for instance for [Spanish (Latin American)](https://www.transifex.com/projects/p/edx-platform/language/es_419/):

1. Configure your  `~/.transifexrc` file:
```
    [https://www.transifex.com]
    hostname = https://www.transifex.com
    username = user
    password = pass
    token =
```
Token is left blank. You have to have permissions for the project (edx-platform) AFAIK - https://www.transifex.com/projects/p/edx-platform/ 
2. Add the language you want in `conf/locale/config`, for instance for the language es_419 (Spanish Latinamerican) it is going to be: 
```
{
 "locales" : ["en", "es_419"],
 "dummy-locale" : "eo"
}
```
3. Configure it in your `lms/env/environment.py`, for instance create one called `dev_es.py` with the following: 
```
from .dev import *

USE_I18N = True
LANGUAGES = ( ('es_419', 'Spanish'), )
TIME_ZONE = 'America/Guayaquil'
LANGUAGE_CODE = 'es_419'
```
4. Execute the following commands in your edx-platform directory with your edx-platform virtualenv, replacing it with the language you want.
```
$ tx pull -l es_419
$ rake i18n:generate
```
5. When you launch your LMS instance you launch it with the environment:
```
$ rake lms[dev_es,0.0.0.0:8000]
```