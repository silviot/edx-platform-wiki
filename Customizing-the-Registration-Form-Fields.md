The [registration form](https://courses.edx.org/register) allows some customization of its fields. While the core fields (email, password, user name and full name) are always mandatory, you can chose whether the other fields should be mandatory, optional, or even not displayed at all. Some additional fields, like city or country, are not displayed in the default form and are available if you want to request them from your users during registration.

This is configurable through the use of the `REGISTRATION_EXTRA_FIELDS` setting, which is a dictionary providing, for each of the fields, one of the following values:

* `required`: to display the field, and make it mandatory
* `optional`: to display the field, and make it non-mandatory
* `hidden`: to not display the field

The default value of the setting is the following:

```python
REGISTRATION_EXTRA_FIELDS = {
    'level_of_education': 'optional',
    'gender': 'optional',
    'year_of_birth': 'optional',
    'mailing_address': 'optional',
    'goals': 'optional',
    'honor_code': 'required',
    'city': 'hidden'
    'country': 'hidden',
}
```