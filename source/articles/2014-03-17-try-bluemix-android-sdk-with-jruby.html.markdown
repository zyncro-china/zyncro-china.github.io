---
title: Try bluemix android SDK with jruby
date: 2014-03-17 08:41 UTC
tags: jruby, ruboto, PaaS, android
---

## Install the SDK

### Create the project

```shell

ruboto gen app -t 14 --package org.ruboto.example.bluemix
```

### Download SDK

The Web page javascript failed to download in Chrome browser, but open the
console, and you can see the download link is:
http://mbaas-catalog.ng.bluemix.net/sdk/ibm-baas-sdk-android.zip

### copy all jars into libs directory

### copy the configuration.json to libs or assets dirtctory?

then add the application id into configuraton.json

### import in the ruby source code

```ruby

#in xx_activity.rb
java_import com.ibm.mobile.services.data.IBMDataException
java_import com.ibm.mobile.services.data.IBMDataObject
java_import com.ibm.mobile.services.data.IBMObjectResult
java_import com.ibm.mobile.services.data.IBMQuery
java_import com.ibm.mobile.services.data.IBMQueryResult
```
### Usage of the classes

```ruby

record = IBMDataObject.new 'record'
```

### Upload data using Rest way

```shell

 curl -v
 "https://mobile.ng.bluemix.net/data/rest/v1/apps/{app_id}/uploads" -X POST -H "Content-Type: application/json" -d '[{"className": "MyPeople","objectId":"123","attributes":{"name":"BBB"}}]''
```

