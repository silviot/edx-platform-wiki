## Overview

This feature allows for user sessions to auto-expire after N seconds of inactivity. This is accomplished via some middleware which notes timestamps on a per request basis. If a request comes in for a user after N seconds has elapsed, the user is logged out and the user is redirected to the default login page.

## To enable

This is an optional feature and it is turned off by default. To turn it on, simply add this to your settings, but you can use your own value for the timeout length:

```
SESSION_INACTIVITY_TIMEOUT_IN_SECONDS = 600  # auto-expire login sessions after 5 minutes of inactivity
```