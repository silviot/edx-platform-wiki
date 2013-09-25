In order to add a new Javascript library, you need to understand a little bit about how require.js is configured. Here's an example:

```js
requirejs.config({
    baseUrl: "/static",
    paths: {
        "domReady": "js/vendor/domReady",
        "jquery": "js/vendor/jquery.min",
        "jquery.ui": "js/vendor/jquery-ui.min",
        "jquery.cookie": "js/vendor/jquery.cookie",
        "jquery.timepicker": "js/vendor/timepicker/jquery.timepicker",
        "datepair": "js/vendor/timepicker/datepair",
        "gettext": "/i18n",
        "mathjax": "https://edx-static.s3.amazonaws.com/mathjax-MathJax-727332c/MathJax.js?config=TeX-MML-AM_HTMLorMML-full",
        // more libraries here... 
    },
    shim: {
        "jquery.ui": {
            deps: ["jquery"],
            exports: "jQuery.ui"
        },
        "jquery.cookie": {
            deps: ["jquery"],
            exports: "jQuery.fn.cookie"
        },
        "mathjax": {
            exports: "MathJax"
        },
        "datepair": {
            deps: ["jquery.ui", "jquery.timepicker"]
        },
        // and so on
    },
    // load these automatically
    deps: ["js/base", "datepair"]
});
```

You can learn more about the [require.js configuration options](http://www.requirejs.org/docs/api.html#config) on the website, but a few things to point out:

  * By convention, libraries are given short names in the `paths` section of the config
  * If a library doesn't register its dependencies with a `define` statement, you can inform require what the library depends on using the `shim` section of the config
  * The libraries listed in the `deps` section will get loaded automatically, first
  * You do not have to list every Javascript file in the `paths` section of the config. If you require a string that doesn't exist in the `paths` map, require will prepend the `baseURL` and append the fixed string `.js`, and fetch that script. That's why you can depend on `js/base` -- require will fetch the file `/static/js/base.js`.

### How to Do It

  1. Download the Javascript library, and save the file in a sensible place in our project (like in `common/static/js/vendor`)
  1. Add the library to the `paths` section of the require.js config
  1. Check the library's dependencies. If the library is already an AMD module, it can define its own dependencies. Otherwise, you'll need to list the dependencies in the `shim` section.
  1. If the library exposes a single object in the namespace, provide the name of that object as a string in the `exports` section of the `shim`. If your library is a jQuery plugin, it will expose an object in the `jQuery.fn` namespace.
  1. Use the library like any other: by including the short name in the list of dependencies passed to the `require` or `define` function when you write your Javascript.