[RequireJS](http://www.requirejs.org/) (also known as just "require") makes Javascript better at handling dependency trees. We will shortly be switching our Studio code over to use require for loading our Javascript. This page exists to give developers a crash-course in what require is and how to use it effectively.

## The Problem

There are three big problems with the class `<script>` tag approach to loading Javascript:

  * Dependency management
  * Lack of isolation (scripts clobber each other)
  * Deployment bundles (scripts running where they shouldn't)

### Dependency Management

Everything in the world depends on jQuery. But some of our scripts also depend on specific jQuery plugins. Some of our scripts rely on Backbone. Some of our scripts depend on MathJax, or CodeMirror, or tinymce, or several other libraries. And good luck if you want to use two libraries that have a namespace conflict: Dojo vs jQuery fighting for the `$` variable, for example. The only solution is to include `<script>` tags in the template for every library that your page depends on, which means they could potentially get fetched multiple times. Bloated, slow, repetitive, not fun.

### Lack of isolation

Javascript is a funny thing: one script can easily affect another. What happens if a script redefines a variable that other scripts are using? For example, `undefined = true` or `$ = alert`. (Don't forget, some of our pages run user-generated Javascript!) More subtly, what happens if one script depends on an earlier script setting or modifying a specific variable, or an element on the page? Changing one script requires changing the other -- which also goes back to our dependency problems.

### Deployment bundles

When we deploy our project, we concatenate and minify our Javascript when it's served in production; this allows clients to retrieve fewer JS files with fewer HTTP roundtrips, which means less overhead.

Here's the problem. Let's say someone writes a script that modifies the elements on a page, intending that script to be loaded only on one specific page (or a small number of specific pages). When we concatenate and minify all of our Javascript files together, all of a sudden that page modification code will load on *every* page of our site. There's no way to take it out, or turn it off and on, without removing it from our space-saving Javascript bundle. Neither option is good: either we pull it out as a separate file, and lose the benefit of our bundling process; or we keep it in the bundle and have it affect pages that we don't want it to affect.

## The Solution

Enter require.js. Require handles all three of these problems with aplomb. The basic idea is to load require via a standard `<script>` tag, and let require handle loading every other script on your website.

### Dependency management

Require exposes a function called `require` (who would have guessed?). Every script *should* use the `require`  function to declare their dependencies. For example:

```js
require(["jquery"], function($) {
  console.log("We have jQuery: ", $);
});
```

As you can see, we wrap all of the content of the file inside an anonymous function, which is then passed to the `require` function, along with a list of dependencies to run that anonymous function. Require.js ensures that the dependencies are loaded and executed before the anonymous function is run, so you don't have to worry about dependency ordering: require handles that for you!

### Isolation

In addition, require will pass values from the dependencies as arguments to the function to be run, so that your function has a guaranteed reference to the dependency. To demonstrate:

The old, broken way:
```html
<script src="jquery.js"></script>
<script>
  $ = "Oops, I'm clobbering $"
  jQuery = "Oops, I'm clobbering jQuery"
</script>
<script>
  console.log("I no longer have a reference to jQuery: ", $)
</script>
```

But require saves the day!
```html
<script src="require.js"></script>
<script>
// load jQuery, and then try to destroy it!
require(["jquery"], function() {
  $ = "Oops, I'm clobbering $"
  jQuery = "Oops, I'm clobbering jQuery"
});
</script>
<script>
// but it wasn't destroyed at all!
require(["jquery"], function($) {
  console.log("require gave me a reference to jQuery, even though the global $ variable has been clobbered: ", $);
});
</script>
```

No more risk of scripts colliding with each other!

### Deployment bundles

Require.js comes with an optimizer called "r.js". It allows us to bundle and minify our Javascript, just the same way that we currently do. However, because all of our Javascript is contained within anonymous functions, the client downloads the Javascript but doesn't execute any of it immediately. Instead, r.js re-writes each the `require` call in the bundle so that require can keep track of where each function came from. For example, if we have these three files:

js/base.js
```js
require(["jquery"], function($) {
  console.log("I run on the base template");
})
```

js/basic.js
```js
require([], function() {
  console.log("I run on a simple page, and have no dependencies");
});
```

js/fancy.js
```js
require(["jquery", "backbone", "tinymce", "jquery.qtip"], function($, Backbone, tinymce) {
  console.log("I run on a fancy page, and I have several dependencies");
});
```

Then r.js will rewrite it to something like:
```js
require("js/base", ["jquery"], function($){console.log("I run on the base template");});
require("js/basic", [], function(){console.log("I run on a simple page, and have no dependencies");});
require("js/fancy", ["jquery", "backbone", "tinymce", "jquery.qtip"], function($, Backbone, tinymce) {console.log("I run on a fancy page, and I have several dependencies");});
```

This way, pages can declare which Javascript they would like to be executed, and require will *only* execute the Javascript that is requested. The big Javascript file is still cached, so it doesn't need to be re-downloaded on subsequent requests, but every page will execute different parts of the Javascript bundle, depending on what is requested.