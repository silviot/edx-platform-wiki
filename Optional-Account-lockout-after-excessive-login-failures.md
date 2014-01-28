## Overview

This features describes a new optional feature in the Open edX platform. It is configurable via a FEATURES flag, which defaults to 'false' (aka off).

This feature will keep track of the number of failed login attempts on a given user's email. If the number of consecutive failed login attempts - without a successful login at some point - reaches a configurable threshold (default 5), then the account will be "locked" for a configurable amount of seconds (15 minutes) which will prevent additional login attempts until this time period has passed.

If a user successfully logs in, all the counter which tracks the number of failed attempts will be reset back to 0.

## How to configure

Simple add this to your settings:

```
FEATURE["ENABLE_MAX_FAILED_LOGIN_ATTEMPTS"] = True

MAX_FAILED_LOGIN_ATTEMPTS_ALLOWED = 10  # defaults to 5, if this is not present
MAX_FAILED_LOGIN_ATTEMPTS_LOCKOUT_PERIOD_SECS = 60 * 60  # defaults to 5 minutes (5 * 60), if this is not present
```