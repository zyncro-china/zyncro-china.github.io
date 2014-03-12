---
title: Update time for raspberry pi
date: 2014-03-13 04:53 UTC
tags: raspberrypi, rtc
---

## The date problem
The raspberry pi doesn't has a real time clock, so every time start the
raspberry pi, we have to resest the time.
## manually set the time from ntp server

```shell
sudo /etc/init.d/ntp stop
sudo ntpdate 1.cn.pool.ntp.org
```
## Solution 2: set the ntp server

```shell

# in /etc/ntp.conf

server 1.cn.pool.ntp.org
server 1.asia.pool.ntp.org
server 2.asia.pool.ntp.org

```

## Solution 3: setup the RTC module

When using the ntp server, it still delay some time after power up the board.
So I try to make use of the RTC module bought online.
