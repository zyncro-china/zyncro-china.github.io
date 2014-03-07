---
title: Improve my reading speed
date: 2014-03-08 15:19 UTC
tags: speedreading, chrome, extension
---

## I heard about this today that can help to increase the reading speed

Today I have found this site [spritz](http://www.spritzinc.com). It has some
theory about [how to improve the reading
speed](http://www.spritzinc.com/the-science/). I have read about some kind of
this theory before. And have installed some iOS app for training speed reading
that based the theory. But most of time I don't have time to do the training
separated. I have to most of content in browser.

## A browser extension for spritz
There is a chrome extension that can display any selected text in the spritz
window at a [github
repository](https://github.com/fantactuka/spritz-chrome-extension).

After installing it, now I can try spritz when reading news, blogs or any
material.

I am looking forward for the spritz to support more languages. They said more
languages are [coming](http://www.spritzinc.com/faq/). 

## Slow down the speed in the extension

The preset reading speed is too fast for me (500 WPM), I have to set a lower
value in the public/javascripts/initialize.js of that extension:

```javascript

panel.setCurrentTextSpeed(200);
```

Let's see if I can increase it in future.

## Add some control to pause and resume

In public/javascripts/initialize.js

```javascript

document.getElementById('pause').addEventListener('click', function() {
  pauseText();
});

document.getElementById('resume').addEventListener('click', function() {
  resumeText();
});

function pauseText(text) {
  window.panel.pauseText();
}
function resumeText(text) {
  window.panel.resumeText();
}

```
