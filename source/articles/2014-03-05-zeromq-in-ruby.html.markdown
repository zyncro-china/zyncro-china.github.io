---
title: ZeroMQ in Ruby
date: 2014-03-05 03:01 UTC
tags: zeromq, ruby, message_queue
---

## 2 ways to write concurrent program

In order to make use of all cpu cores and increase the capability, we need to
write concurrent programs that can communicate with each other.

There are 2 different communication options:

1. inter-thread
1. inter-process

When using inter-thread option, the thread can share the same memory space in
the process. But shared data in different running thread can also easily cause some bug that is hard to find. It is also limited on the same host. So it is not scalable.

Inter-process takes more memory space, but it has no disadvantage as the
inter-thread option. So inter-process option is preferred in most of the cases.

## Inter-Process communication

on unix system, There are at least 2 ways to implement inter-process
communication:

1. pipe
1. socket(tcp, udp, unix)


In the shell, using | can pass the data between different program.

And mkfifo can also create a named pipe(fifo). It can easily be used to shared
data between unrelated process.

TCP socket is reliable and UDP is not reliable. UNIX socket is only available
on the same host.

## ZeroMQ and its communication models

ZeroMQ provides a wrapper for inter-process(also include inter-thread) communication.

The underlying implementation includes:

1. tcp
  It use TCP Socket, and don't tell the port number because there is no
  authentication
1. ipc
  It use UNIX Socket.
1. inproc
  It avoid manually shared data between different threads.
1. multicast
  Implemented in UDP multicast, but some router could ban this.

The ZeroMQ communication models includes:

1. REQ/REP
1. PUB/SUB
1. PUSH/PULL
1. PAIR

## Using ZeroMQ in Ruby

### Install
gem install zmq

### Usage
For the REQ/REP model:

```ruby

# rep.rb

require 'zmq'

context = ZMQ::Context.new
socket = context.socket ZMQ::REP
socket.bind 'tcp://127.0.0.1:5000'

loop do
  msg = socket.recv
  puts "Got: ", msg, "\n"
  socket.send msg
end

# req.rb

require 'zmq'

context = ZMQ::Context.new
socket = context.socket ZMQ::REQ
socket.connect 'tcp://127.0.0.1:5000'

(1..3).each do |i|
  socket.send("msg %s" % i)
  msg = socket.recv
  puts "Replied: ", msg, "\n"
end
puts 'the end'
socket.close

```

Remember to call socket.close before exit the program.

Using * instead of 127.0.0.1 can accept all hosts

The client and server don't have to start in order.

### Create a Chatting Service

Using Push/Pull and Pub/Sub models, we can create a chatting service.
We need 2 sockets:
1. message-queue
1. message-display

The chatting client push the new message to the message-queue socket, and subscribe a message-display socket.

The chatting server pull from the message-queue socket, then publish to the message-display socket
