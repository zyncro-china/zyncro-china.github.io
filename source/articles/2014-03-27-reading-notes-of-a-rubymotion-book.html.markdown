---
title: Reading notes of a rubymotion book
date: 2014-03-27 06:38 UTC
tags: rubymotion
---
### Use NSLog instead of puts
Because puts wouldn't output to device logs

### RubyMotion provide help for 3 things
1. flexible workflow(editor, command line, repl)
1. use ruby language
1. skip the large set of new API have to learn

### Value Object
There are 4 value object classes
1. NSString
1. NSNumber
1. NSDate
1. NSData( for byte buffers)

### Major design patterns in iOS development
1. MVC(the main difference from rails is that working with user interactions
   and system events instead of HTTP requests)
1. Delegation
1. Protocols
1. Notification Center(Pub/Sub)
1. Target-Action
1. Key-Value Observing(more Observers)
1. View Hierarchy
1. responder Chain
1. Receptionist

### MVC pattern
1. difference between iOS and Rails
The main changes to controller and view layers in relation to events
1. models are not alwasy database based
1. view objects are inherit from UIView or NSView, and responsible for drawing
   theirselves and subviews. In the case of controls, handling user interaction
   and input, then tell the controller about it.
