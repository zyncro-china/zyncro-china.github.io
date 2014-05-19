---
title: Hacking simplewebrtc.js to change the video resolution
date: 2014-02-25 07:37 UTC
tags: webrtc
---

## Introduction webrtc and simplewebrtc.js
I am very exciting about WebRTC technology. It makes the browser can provide
some peer to peer functionalities that can't be thought before.

The only limitation now is that only a few mainframe browsers support WebRTC,
and they use different API.

I think this is where simplewebrtc.js can help the WebRTC development easier. By
using the unified interface, developer don't have to handle the difference
between browsers.

I hope this library will not be too complicated, as its
name suggests. So that I can modify when I need, without having to read many
code that I don't use.

## The SimpleWebRTC can't customize the video resolution
The simplewebrtc doesn't support changing constraints.

```js

SimpleWebRTC.prototype.startLocalVideo = function () {
    var self = this;
    this.webrtc.startLocalMedia(null, function (err, stream) {
        if (err) {
            self.emit(err);
        } else {
            attachMediaStream(stream, self.getLocalVideoContainer(), {muted: true, mirror: true});
        }
    });
};
```
In order to pass in the constraints, I override this function(I write the app
in coffeescript):

```coffeescript

SimpleWebRTC.prototype.startLocalVideo = ->
  self = this
  this.config.constraints ||= {video: true, audio: true}
  this.webrtc.startLocalMedia this.config.constraints, (err, stream)->
    if err
      self.emit(err)
    else
      attachMediaStream(stream,
        self.getLocalVideoContainer(),
        {muted: true, mirror: true})
```
Then we can pass the constraints in the constructor:

```coffeescript
webrtc = new SimpleWebRTC
  localVideoEl: 'localVideo'
  remoteVideosEl: 'remotes'
  autoRequestMedia: true
  debug: true
  detectSpeakingEvents: true
  autoAdjustMic: false
  constraints:
    audio: true
    video:
      mandatory:
        maxWidth: 320
        maxHeight: 180
```

## Update
The SimpleWebRTC github commit on Mar 23 fix this problem. this hack is not
needed anymore.
