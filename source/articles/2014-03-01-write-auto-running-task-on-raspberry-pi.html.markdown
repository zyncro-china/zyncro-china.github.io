---
title: Write auto running task on raspberry pi
date: 2014-03-01 14:26 UTC
tags: raspberrypi, upstart, ruby
---

### A cheap board running complete linux OS
Not like Arduino, the Raspberry Pi board can run a full linux OS and also have
GPIO pins, so there are more thing could be done on the Raspberry Pi. Now
I need to find out how to run some task automatically when the board boots.

### Install Upstart package
When install upstart, it ask some message that make me worry:

```

You are about to do something potentially harmful.
To continue type in the phrase 'Yes, do as I say!'


...
Installing new version of config file /etc/modprobe.d/fbdev-blacklist.conf ...
Installing new version of config file /etc/init.d/udev ...
initctl: Unable to connect to Upstart: Failed to connect to socket /com/ubuntu/upstart: Connection refused
[ ok ] Stopping the hotplug events dispatcher: udevd.
[ ok ] Starting the hotplug events dispatcher: udevd.
update-initramfs: deferring update (trigger activated)
Setting up libjson0:armhf (0.10-1.2) ...
Setting up upstart (1.6.1-1) ...
Processing triggers for initramfs-tools ...
```

I follow the instruments and reboot. Then it can't boot to the login prompt again.

After re-write the new image to the SDCard, and the above process runs fine.

### Write a new upstart task

#### check the syntax
1. init-checkconf -d myservice.conf

Notes: a work around when the checking meets error: "ERROR: failed to ask
Upstart to check conf file"

[init-checkconf bug](https://bugs.launchpad.net/upstart/+bug/881885)

```

sudo apt-get install dbus-x11
eval `dbus-launch --auto-syntax`
```

#### list the service
initctl list
