---
title: 在android上创建chrome app
date: 2014-02-01 15:35 UTC
tags: mobile, android, chrome, phonegap
---
# Why not just using phonegap to build the mobile app

Because setup phonegap toolchain takes more time, while [cca](https://github.com/MobileChromeApps/mobile-chrome-apps) is much faster. Further more, it can make use of chrome api.
# installation
sudo npm install -g cca

# create project
cca create test --android

# deploy on the phone
* cca build android
* cca run android --device

