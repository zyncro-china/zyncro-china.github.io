---
title: use bootstrap and sass in a semantic way
date: 2014-02-24 03:46 UTC
tags: bootstrap, sass
---

## Using twitter bootstrap directly means mix things together
When developing the web page, using bootstrap could accelerate the process. But
mixing the content and the display style is not a very good solution.

So we try to use SASS to isolate the styles, and make the html more semantic.

## SASS features that helps
[mixins](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#mixins)
and
[extend](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#extend)
allow you to define styles that can be re-used throughout the stylesheet
without needing to resort to non-semantic classes like .float-left

## Convert an example that use bootstrap style to semantic html
The original index.haml:

```haml

#main
  %a.btn.btn-primary Press Me

```

using extend, we use more meaningful class: "highlight", and convert the above code into following haml:

```haml
#main
  %a.highlight

```

and the scss file: 

```scss

@import "variables";
@import "bootstrap";

#main {
  a.highlight {
    @extend .btn;
    @extend .btn-primary;
  }
}
```

It seems like that we have to write more lines, but isolating things will pay
the costs in the long term when the project grow bigger.
