---
title: Write presentation in markdown and hosted on heroku
date: 2014-03-07 14:34 UTC
tags: markdown, heroku, presentation, reveal.js
---

## Write docs in Markdown

As more and more docs and email can be written in markdown format, It is time
to try it in the slides/presentation.

## Reveal.js

Seems reveal.js is very easy to use, I try to use it to write slides, and
I hope it can be hosted on heroku, so I can use it no matter what devices I am
using, and where I am.

## Installation

### install reveal.js

```shell

git clone git@github.com:MichaelBitard/revealjs_heroku.git

```

### use express to host

```javascript
//web.js

var express = require("express");

var app = express();
app.use(express.logger());
app.use("/", express.static(__dirname));

var port = process.env.PORT || 5000;
app.listen(port, function() {
  console.log("Listening on " + port);
});

```

### add heroku Procfile

```shell

web: node web.js

```

### add http basic auth to protect the content

```javascript

var auth = require('http-auth');
var basic = auth.basic({
    realm: "Zyncro China Presentation.",
    file: __dirname + "/data/users.htpasswd" // gevorg:gpass, Sarah:testpass ...
});

app.use(auth.connect(basic));
```
### troubleshooting

If express start failed(call: node web.js), then select the express version:
3.1.x
