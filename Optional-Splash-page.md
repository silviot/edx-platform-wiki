This feature allows to display a small marketing landing page upon the first visit of a visitor, protect an
alpha website from the public eye, make an announcement, etc.

It checks a given cookie in incoming requests, and redirects users to a configured splash screen URL if they don't have the proper cookie set. It's then the responsibility of the page the user is redirected to to set the proper cookie before sending the user back to the LMS.

Configuration
=============

First, enable the feature:

```python
FEATURES['ENABLE_SPLASH_SCREEN'] = False
```

Then configure the redirection:

```python
############################### Splash screen ####################################

SPLASH_SCREEN_COOKIE_NAME = 'edx_splash_screen'

# The user cookie value must match one of the values to not be redirected to the
# splash screen URL
SPLASH_SCREEN_COOKIE_ALLOWED_VALUES = ['seen']

# Users which should never be redirected (usernames)
SPLASH_SCREEN_UNAFFECTED_USERS = []

# The URL the users should be redirected to when they don't have the right cookie
SPLASH_SCREEN_REDIRECT_URL = 'http://edx.org'
```

Tip: You can set one or several values in `SPLASH_SCREEN_COOKIE_ALLOWED_VALUES`. You can use this to implement a lightweight auth on a semi-private alpha site. The alpha site can ask for a password, store the value in the cookie, and let the LMS check if the value matches one of several allowed credentials. This is not a secure approach, but it can be sufficient if you simply want to keep the instance semi-private until it's ready for prime time.