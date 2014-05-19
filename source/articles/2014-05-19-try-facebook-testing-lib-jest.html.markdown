---
title: Try facebook testing lib Jest
date: 2014-05-19 08:56 UTC
tags: javascript, coffeescript, testing
---

## The new testing framework from facebook
Facebook released their new testing framework:
[Jest](http://facebook.github.io/jest/)

It seems to be very nice that can help you to mock the dependencies
automatically:

> In order to write an effective unit test, you want to be able to isolate a unit
> of code and test only that unit – nothing else. It is fairly common and good
> practice to consider a module such a unit, and this is where Jest excels. Jest
> makes isolating a module from its dependencies extremely easy by automatically
> generating mocks for each of the module's depenedencies and providing those
> mocks (rather than the real dependency modules) by default.

Jest actually inspect the file that the code under test are requiring, then
mock every properties of the required object, and generate the mocked function.

In this way, we don't have to manually generate mocks for testing. This is
a huge benefits when writteing tests.

## writing tests in coffeescript 

I also like to use coffeescript these days. But everytime, it seems that I need
to pay more efforts to make it works, especially when the project is mainly
developed only using javascript.

Jest have a page explained [how to use
coffeescript](http://facebook.github.io/jest/docs/tutorial-coffeescript.html#content)

Just added following contents:

```javascript
// package.json
  "dependencies": {
    "coffee-script": "*"
  },
  "jest": {
    "scriptPreprocessor": "preprocessor.js",
    "testFileExtensions": ["coffee", "js"]
  }
// preprocessor.js
var coffee = require('coffee-script');

module.exports = {
  process: function(src, path) {
    if (path.match(/\.coffee$/)) {
      return coffee.compile(src, {'bare': true});
    }
    return src;
  }
};
```

I have used that way and it can't work, it always return following result:

```bash
 Found 0 matching tests...
```

After some code reading, I found the TestRunner.js in Jest may construct the RegExp incorrectly. For js and coffeescript, it generate following RegExp:

```
//__tests__/.*\.(js\|coffee)$/
```

But it should be 

```
//__tests__/.*\.(js|coffee)$/
```

After testing, and submit the patch to Jest's github [pullrequest](https://github.com/facebook/jest/pull/34). Then I found that I also need to sign the CLA(Contributor License Agreement) to submit the code to facebook. And that CLA link is blocked in China. Fortunately I have a VPN. So I think there will be very little patch from China.

Now I can continue trying Jest after spending half of the day to figure out this problem.
