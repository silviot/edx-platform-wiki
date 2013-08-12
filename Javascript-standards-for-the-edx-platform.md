## Google Standards
In the [Google JavaScript Style Guide](http://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml) the section [JavaScript Language Rules](http://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml#JavaScript_Language_Rules) has very reasonable "tips" on how one should and shouldn't write JavaScript code. Since we don't want to re-invent the wheel and come up with and maintain our own unique set of standards, we have adopted these as our go-to standards.

## Code Style
Use 4 spaces for indenting
```
function renderElements(state) {
    var youTubeId;

    if (state.videoType === 'html5') {
        state.videoPlayer.player = new HTML5Video.Player(state.el, {
            playerVars: state.videoPlayer.playerVars,
            videoSources: state.html5Sources,
            events: {
                onReady: state.videoPlayer.onReady,
                onStateChange: state.videoPlayer.onStateChange
            }
        });
    }
}
```

## Linting
Use [JSHint](http://www.jshint.com/about/) with the following settings:
TDB

## Good Practices

All local variables in a function should be declared first, before any code statements.
Even if initialization happens later on, the variable should be declared. This is benefiter for quickly checking which variables are local and which are defined further up the scope chain (or are global). One doesn't have to skim through the entire function body searching for the var keyword to discover local variable declarations - they are all in one place at the top. This also follows the logic of hoisting in JavaScript. By placing all variable declarations at the top, you are always aware of the fact that all variables are declared at the time of function's invocation, and by default assigned the value of undefined. Even if the variable definition happens somewhere in the middle of the function's body, the variable declaration is silently "hoisted" to the top of the function by the compiler.

Think carefully when to use function declaration and when to use function expression.
If the function will be invoked throughout the code, then it is probably a safe bet to use a function declaration. Due to hoisting, it will be immediately available (either globally, or in the scope of the function), even if declared at the end of the source file or function body. However, if the function will be used in some cases, then a function expression is more suitable. The function will be "created" only when AND IF the compiler execution flow reaches the function expression statement. Memory will be allocated only when necessary. 
When developing a new applet, internal function API is generally hidden from everyone else.
This means that one is pretty sure with what parameter's and their types functions will be invoked with. This means that it is redundant to do type checking or other double-safety things on function's parameters. However, in the initial stage of development, it is good to check. For this it is best to write tests (see Jasmine, unit testing).

When creating an object and populating it with properties, the names of the properties should be surrounded by quotes only if they include some unsupported symbols. Otherwise, for code consistency, leave off the quotes (single or double).

Sometimes it is very tempting to test if an object really does contain a property by using the Object.hasOwnProperty() method. This is unnecessary, and should be done only in the case when it has been confirmed that the prototype chain of the object also has a property with the same name.

## Testing
The front-end is lacking in unit testing. There is a Jasmine based testing foundation already setup, however, [Jasmine](http://pivotal.github.io/jasmine/) is not used thoroughly as desired. Include at least some starting tests for each JavaScript applet/project.
[This video of GTAC 2013 Day 2 Keynote: Testable JavaScript](http://www.youtube.com/watch?feature=player_embedded&v=JjqKQ8ezwKQ) gives a lot of good advice in general about writing testable code, with specific examples of how to do it in JavaScript.  If we wrote JavaScript the way this presentation suggests (creating clear interfaces, then testing those interfaces), writing Jasmine tests would be a lot easier.

## Documentation
Some programmers document their code, others do not. The world of JavaScript has a nifty informal documentation spec JSDoc. It is currently at version 3. Complete reference is located at [GitHub page](https://github.com/jsdoc3/jsdoc), and [official site](http://usejsdoc.org/).

The following is a list of useful JSDoc version 3 tags:
- [@access](http://usejsdoc.org/tags-access.html) Specify the access level of this member - private, public, or protected.
- [@author](http://usejsdoc.org/tags-author.html) Identify the author of an item.
- [@callback](http://usejsdoc.org/tags-callback.html) Document a callback function.
- [@constant](http://usejsdoc.org/tags-constant.html) Document an object as a constant.
- [@constructor](http://usejsdoc.org/tags-constructor.html) This function is intended to be called with the "new" keyword.
- [@default](http://usejsdoc.org/tags-default.html) Document the default value.
- [@desc](http://usejsdoc.org/tags-description.html) Describe a symbol.
- [@example](http://usejsdoc.org/tags-example.html) Provide an example of how to use a documented item.
- [@exports](http://usejsdoc.org/tags-exports.html) Identify the member that is exported by a JavaScript module.
- [@external](http://usejsdoc.org/tags-external.html) Document an external class/namespace/module.
- [@file](http://usejsdoc.org/tags-file.html) Describe a file.
- [@fires](http://usejsdoc.org/tags-fires.html) Describe the events this method may fire.
- [@global](http://usejsdoc.org/tags-global.html) Document a global object.
- [@link](http://usejsdoc.org/tags-link.html) Inline tag - create a link.
- [@member](http://usejsdoc.org/tags-member.html) Document a member.
- [@method](http://usejsdoc.org/tags-method.html) Describe a method or function.
- [@module](http://usejsdoc.org/tags-module.html) Document a JavaScript module.
- [@namespace](http://usejsdoc.org/tags-namespace.html) Document a namespace object.
- [@param](http://usejsdoc.org/tags-param.html) Document the parameter to a function.
- [@private](http://usejsdoc.org/tags-private.html) This symbol is meant to be private.
- [@property](http://usejsdoc.org/tags-property.html) Document a property of an object.
- [@protected](http://usejsdoc.org/tags-protected.html) This member is meant to be protected.
- [@public](http://usejsdoc.org/tags-public.html) This symbol is meant to be public.
- [@readonly](http://usejsdoc.org/tags-readonly.html) This symbol is meant to be read-only.
- [@requires](http://usejsdoc.org/tags-requires.html) This file requires a JavaScript module.
- [@returns](http://usejsdoc.org/tags-returns.html) Document the return value of a function.
- [@see](http://usejsdoc.org/tags-see.html) Refer to some other documentation for more information.
- [@summary](http://usejsdoc.org/tags-summary.html) A shorter version of the full description.
- [@this](http://usejsdoc.org/tags-this.html) What does the 'this' keyword refer to here?
- [@throws](http://usejsdoc.org/tags-throws.html) Describe what errors could be thrown.
- [@todo](http://usejsdoc.org/tags-todo.html) Document tasks to be completed.
- [@type](http://usejsdoc.org/tags-type.html) Document the type of an object.
- [@version](http://usejsdoc.org/tags-version.html) Documents the version number of an item.