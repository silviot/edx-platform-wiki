## Overview

Open edX installations can configure how complex user passwords are required to be. These rules will be enforced whenever a user is setting a new password (e.g. on account creation or password reset).

If you change these options, the new rules will only be applied to future account creations or password resets. Existing passwords will not be affected.

The default settings allow any password between 2 and 75 characters long.

## Options

* **PASSWORD_MIN_LENGTH**: The required minimum password character length (each Unicode code point is one character). Defaults to 2.
* **PASSWORD_MAX_LENGTH**: The maximum allowed password character length (each Unicode code point is one character). Defaults to 75.

The rest of these options are gated behind a ENFORCE_PASSWORD_POLICY feature flag that defaults to false. So if you want to use these, you also have to set `FEATURES["ENFORCE_PASSWORD_POLICY"] = True`. 

* **PASSWORD_DICTIONARY**: A list of words that the password cannot be similar to. The exact similarity is controlled by the next option below.  _Both_ options must be set for the dictionary to be used.
* **PASSWORD_DICTIONARY_EDIT_DISTANCE_THRESHOLD**: How different the user password is allowed to be from any of the words in the above dictionary. This option is based on Levenshtein distance.

* **PASSWORD_COMPLEXITY**: This is a set of sub-options that control the complexity requirements (things like "must contain a symbol" etc).
  - **UPPER**: The number of unique uppercase characters that must appear in the password.
  - **LOWER**: The number of unique lowercase characters that must appear in the password.
  - **ALPHABETIC**: The number of letter characters that must appear in the password. _New in Hawthorn._
  - **DIGITS**: The number of unique 0-9 characters that must appear in the password.
  - **NUMERIC**: The number of number characters that must appear in the password. (This is more expansive than DIGITS, allowing the full range of Unicode numbers.) _New in Hawthorn._
  - **PUNCTUATION**: The number of unique ASCII punctuation characters that must appear in the password.
  - **NON ASCII**: The number of unique non-ASCII characters that must appear in the password.
  - **WORDS**: The number of unique words (separated by whitespace) that must appear in the password.

## Example

```
PASSWORD_MIN_LENGTH = 8
PASSWORD_MAX_LENGTH = 100

FEATURES['ENFORCE_PASSWORD_POLICY'] = True

PASSWORD_COMPLEXITY = {
    'ALPHABETIC': 1,
    'NUMERIC': 1,
}
PASSWORD_DICTIONARY = ['password', 'abcdefgh']
PASSWORD_DICTIONARY_EDIT_DISTANCE_THRESHOLD = 1
```