## Overview

This feature is to be able to enforce Open edX account password policies, via reuse of the django-password project. This feature is enabled via a `FEATURES["ENFORCE_PASSWORD_POLICY"] = True` switch. Default is False (aka don't use this feature).

The various password policies to control are:

- password min length
- password max length
- min # of Digits required in password
- min # of uppercase characters required in password
- min # of lowercase characters required in password
- min # of punctuation characters required in password
- similarity to well known passwords (e.g. don't allow 'abcdef' or 'password')

IMPORTANT: Right now this enforcement only happens on new account creation. However similar policies need to be enforced on password reset workflows. Right now it appears that we are using an out-of-the-box django password reset workflow, which will have to be changed. This work is TBD. Also, note, that existing passwords will be 'grandfathered in' - it will not go through existing user accounts and apply that enforcement.

## Example on how to Configure to use Policies

```
FEATURES['ENFORCE_PASSWORD_POLICY'] = True

PASSWORD_MIN_LENGTH = 4
PASSWORD_MAX_LENGTH = 12
PASSWORD_COMPLEXITY = {
    'UPPER': 2,
    'LOWER': 2,
    'PUNCTUATION': 2,
    'DIGITS': 2
}
# Not the best setting name, but this is the list of passwords to not allow similarity to
PASSWORD_DICTIONARY = ['password', 'abcdef']
```