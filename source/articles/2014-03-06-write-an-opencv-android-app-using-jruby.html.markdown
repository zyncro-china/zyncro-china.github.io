---
title: Write an opencv android app using jruby
date: 2014-03-06 07:25 UTC
tags: opencv, android, rubuto, jruby
---

## Download opencv lib
Download lib from http://sourceforge.net/projects/opencvlibrary/files/opencv-android/2.4.8/
Unzip the files

## Setup ruboto project

### use Ruboto CLI to create a new project

```shell

ruboto gen app -t 15 --package org.ruboto.example.opencv

```

### Add the opencv project into the project.properties

```java

android.library.reference.1=../OpenCV-2.4.8-android-sdk/sdk/java 
```

### Add following import lines into the Activity file:

```ruby

import 'org.opencv.android.BaseLoaderCallback'
import 'org.opencv.android.CameraBridgeViewBase.CvCameraViewFrame'
import 'org.opencv.android.LoaderCallbackInterface'
import 'org.opencv.android.OpenCVLoader'

```

Now you can run following command to build the app and check if the OpenCV Lib
is imported correctly.

```shell

rake install start
```

### Start from here to write the awesome OpenCV App using ruby
Now we can start to write the app.

### Trouble Shooting

#### missing build.xml

Seems the new android version require build.xml, so just copy this file from
other project and change the project name in it.

#### Failure [INSTALL_FAILED_ALREADY_EXISTS]

use the adb command to delete the existed app:

```shell

adb uninstall org.ruboto.ruanwz.opencv_demo
```
