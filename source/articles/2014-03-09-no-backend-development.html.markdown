---
title: No Backend Development
date: 2014-03-09 02:38 UTC
tags: BaaS, nodejs, PaaS, heroku
---

## Backend As A Service

Recently I have used some Backend As A Service(BaaS) platform to make the
development faster. How can it be faster than not doing. So for some very small
demos, I tried firebase. It seems good. Only one thing is missing, it dont
support to store an array. So everything must be an object.

## Find an alternative solution

In order to achieve the following goal, I want to find another firebase like
platform.

1. Support array
1. Can be privately hosted in own machines
1. Can also hosted on some popular PaaS platform like heroku
1. Open source project that we can dig into the code when we need

I found the deployd is good, and although there is some issues when I issue
some commands in the console. But other things works well at the first glance.

## Deploy on heroku

Heroku is so wonderful platform that we can verify some idea very quickly by
deploy a demo without any budget.

Following is how to deploy deployd on heroku

### setup a new heroku app

```shell

heroku apps:create myawesomedeployd

```

Next, add mongodb resource

```shell

 heroku addons:add mongolab
```

Add heroku config files and the node.js script

```shell

# Procfile
web: node server

```

```javascript

// package.json 
{
  "name": "app_name",
  "version": "0.0.1",
  "description": "Super awesome app",
  "keywords": [],
  "homepage": "",
  "author": "You!",
  "contributors": [],
  "dependencies": {
    "deployd": ">= 0"
  },
  "scripts": {
    "start": "node server"
  },
  "engines": {
    "node": "0.8.x",
    "npm":  "1.2.x"
  },
  "license": "MIT"
}


// server.js 

// require deployd
var deployd = require('deployd');
// configure database etc. 
var server = deployd({
  port: process.env.PORT || 5000,
  env: 'production',
  db: {
    host: 'host.mongolab.com',
    port: 33559,
    name: 'heroku_app',
    credentials: {
      username: 'heroku_app',
      password: 'password'
    }
  }
});

// heroku requires these settings for sockets to work
server.sockets.manager.settings.transports = ["xhr-polling"];

// start the server
server.listen();

// debug
server.on('listening', function() {
  console.log("Server is listening on port: " + process.env.PORT);
});

// Deployd requires this
server.on('error', function(err) {
  console.error(err);
  process.nextTick(function() { // Give the server a chance to return an error
    process.exit();
  });
});


```

### generate the keys for deployd

```shell

mkdir .dpd
touch .dpd/.gitemptydir
git add .dpd/.gitemptydir
dpd keygen
dpd showkey

```

### create a resource directory to avoid node.js error

When running deployd, it could report:

```shell

deployd Error: ENOENT, readdir 'resources'
```

Just need to commit an empty directory to avoid this error

```shell

mkdir resources
touch resources/.gitemptydir
git add *
git reset node_modules
git commit -m 'add resources/\n'

```

### Deploy to Heroku

Now just push to heroku

```shell

git push origin master
```
Then access from http://herokuhost.com/dashboard

### Connect to mongolab to check the data:

```shell

mongo -u heroku_app2239 -p password_str --port 3355 --host
ds033.mongolab.com heroku_app2239
```
