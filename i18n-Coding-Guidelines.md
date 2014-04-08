Developers are expected to internationalize (i18n) all user-facing strings.

Detailed instructions, including how to i18n and provide `Translators:` comments in different types of files (Python, Mako, Javascript, etc) can be found here: https://github.com/edx/edx-platform/blob/master/docs/en_us/developers/source/i18n.rst

Additionally, here are some general guidelines you should always follow:

* Always provide context in your strings - this means using well-named placeholder variables over positional variables (`_("Showing grades for {username}").format(username=username)` as opposed to `_("Showing grades for {0}").format(username)`)
* Context should also be provided via `# Translators:` comments. These comments can explain in detail how a word is used. This is especially important for shorter phrases that may have a few different interpretations.
* In Python, use `.format` rather than `%` for variable substitution
* Try to keep as much HTML markup out of the string as possible. For example, do: `"<em>" + _("Sign up here!") + "</em>"` as opposed to `_("<em>Sign up here!</em>")`.


Overall, when i18n, try to:
--------
   1. Shove HTML tags into other variables rather than including them in the strings

   2. Use string concatenation to avoid i18ning parts of strings unnecessarily (eg, to avoid i18ning formatting around text)

   3. A valuable tool is that you can call _() within a .format() clause, allowing you to just i18n the text of a string that has a bunch of other weird placeholders or markup.

### Examples

**Lesson 1**. {span_start}{something}{span_end} - why are there 3 tags here? This can be seriously messed up by translators. Put everything into {something} and just have one tag.

Example:

Before:
```
<h2>${_('{span_start}{platform_name}{span_end} Help').format(
    span_start='<span class="edx">', span_end='</span>', platform_name=platform_name
)}</h2>
```

After:
```
<h2>${_('{platform_name} Help').format(
    platform_name=u'<span class="edx">{}</span>'.format(platform_name)  # Make sure this is u"" for Unicode platform names
)}</h2>
```

**Lesson 2.** Similar to Lesson 1, this involves <mailto> tags and is super common in the LMS. Do not do this!

Before: 
```
_("Please email us at <a href=\"mailto:{tech_support_email}\">{tech_support_email}</a> to report any problems or downtime.").format(
    tech_support_email=settings.TECH_SUPPORT_EMAIL)
)
```
After: 
```
_("Please email us at {tech_support_email} to "
 "report any problems or downtime.").format(
     tech_support_email=u"<a href=\"mailto:{0}\">{0}</a>".format(settings.TECH_SUPPORT_EMAIL)
)
```

**Lesson 3.** Also similar to Lesson 1 - tags around a variable should be subsumed into that variable.

Before:

`_("<em>{platform_name}</em> servers").format(platform_name=settings.PLATFORM_NAME)`

After:
```
_("{platform_name} servers").format(
    platform_name=u"<em>{}</em>".format(settings.PLATFORM_NAME)  # Make sure this is u"" for Unicode platform names
)
```

**Lesson 4.** If an element *MUST* be present in the string, and does not require translation, do not put it in the translation string.

Example: When registering a new account, required fields are marked with an asterisk.

Before: `${_('E-mail *')}`

After: `${_('E-mail')} + ' *'`


Similar to this, break up your strings if you have parts - particularly parts with lots of interpolated variables!! - that don't need translation. For example:


Before: `gettext("%(voteNum)s%(startSrSpan)s vote (click to vote)%(endSrSpan)s")`



After: `"%(voteNum)s%(startSrSpan)s " + gettext("vote (click to vote)") + "%(endSrSpan)s"`



**Lesson 5.** You can actually translate things within the ".format" clause, which can be helpful:


Before: `${_("{a_start}Return to where you left off{a_end}").format(`


After:
```
${"{a_start}{text}{a_end}".format(
    text=_("Return to where you left off"),
```