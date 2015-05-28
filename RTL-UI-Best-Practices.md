This page captures the considerations and best-practices for changing the edX platform UI to read from right to left for languages that are read from right to left.

## Flipping the Sass
We are using Bi-App Sass to help flip our UI for languages that read right to left. The primary css properties that are involved in flipping are **float, margin, padding, text-align, positioning, and borders**. When you are using these properties in the edX platform, you should check whether it will need to be flipped for right-to-left languages. If so, use the bi-app mixins instead of the regular property: value syntax. For instructions on usage, check the bi-app readme: https://github.com/anasnakawa/bi-app-sass/blob/master/README.md

### Testing Your Feature

To test, spin up your local dev environment and visit the webpage of your feature. In a different browser, or an incognito window, visit the feature again (log in as a different user, if logins are needed). In the second view, append the following string to the end of the url: ?preview-lang=ar

This will switch your session's language to Arabic, which is a RTL language (if Arabic is released on your dev environment, instead just switch your user's profile language to Arabic). Navigate your feature in the English webpage and the Arabic webpage simultaneously. You should see that all elements in the Arabic version mirror the English version.

Note that certain page elements may not look right because they are controlled by the browser. For the best testing, switch your browser's language to Arabic as well.

Reach out to us on the edx-code mailing list if you have any questions about testing.

## Other Considerations
### Semantics
An HTML document is a stand alone piece of content that should have a language and character set attribute set early in the DOM. In addition, adding an attribute defining the language direction should be part of the DOM, either on the document as a whole, or on the specific pieces of content that read right to left or left to right. Changing the display language should change any of those attributes that apply. And, as always, structure your HTML in a logical order (main content first, supplementary content following) and with proper semantic structure (sections, headers, etc) to make it readable and adjustable for various contexts. (Turn off css to check that the document flow is logical.)

_Dev actions: Structure your HTML semantically; set language and character attributes early in the DOM_

### Visual Layout
In addition to indicating that the raw document content should be read right to left, the visual layout of the page should change to accommodate a change of language and language direction: content written in RTL languages should appear right aligned (and vice versa). For full page changes, main content should appear to the right, sidebars of supplementary content should shift from right to left, "previous" arrows should point right (and "next" to the left). Form field content should fill from right to left as well. 

_Dev actions: Visually check that layout flips_

### HTML
The W3C recommends adding the language direction attribute (dir="rtl") on the root (html) element to define the language direction of the document. Because some browsers incorrectly switch the location of the scroll bar to the left as well, current best-practice is to add the dir="rtl" to the `<head>` tag and to a `<div>` wrapping the whole page to flip the entire document (which is an alternate recommendation by W3C) to ensure that the scrollbar isn't flipped.

_Dev actions: Add dir="rtl" to head element and highest content-wrapping div element; add dir="ltr" to any URL or file location text elements_

### SASS/CSS
There are a variety of ways to use css to change the direction of text, but some are better than others, and following some best-practices can help alleviate trouble later. Most importantly, using "display: inline-block;" correctly when laying out pages can allow the document or element direction property on the document or specific html elements (dir="rtl") to reverse the layout as well as the content. Also, setting equal margin/padding on both sides of elements when possible is better than having to explicitly flip. 
(thanks to [Integralist](https://gist.github.com/Integralist/7269907) for the gist of this!)

_Dev actions: Flip as little as possible; use good practices to set things up for clean flips_

