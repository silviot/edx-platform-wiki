[XBlock](https://github.com/edx/XBlock) is not fully implemented, so new courseware content types still need to be added as XModules if you want to run them on edx-platform today. If you're writing a new XModule and want to minimize transition pain to XBlock, keep the following in mind (cribbed from an email from cpennington):

1. Don't use `render_template` from system (use your own templating instead)
2. Use as little as possible from system in general (since most of that isn't standardized in XBlock yet)
3. Write handle ajax as a dispatcher to functions with the same name as the `handle_ajax` action, and tag those with `XBlock.json_handler`
4. Don't write an XModule that overrides `get_display_items` (since XBlock doesn't have a similar facility)
