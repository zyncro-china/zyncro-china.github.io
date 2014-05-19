---
title: create iOS app using Promotion
date: 2014-03-24 09:10 UTC
tags: iOS, rubymotion, ProMotion, teacup, mobile
---

## The Goal

Create an iOS app as quickly as we can.

## Project Setup

### Create a new app
```shell

promotion new membershipapp
```

### run tests automatically

```shell

gem install when-files-change
when-files-change "clear && bundle exec rake spec"
```

## write test cases

The Promotion TDD framework will instantiate @controller and add a test @app by tests method, we need to call the Promotion::Screen.new and want to use screen instead of controller:

```ruby
#home_screen_spec.rb

describe HomeScreen do
  tests HomeScreen

  def controller
    @controller ||= HomeScreen.new
  end
  alias :screen :controller

  it "is a TableScreen" do
    screen.should.be.kind_of(PM::TableScreen)
  end
end
```

## create a basic tab screen
The Tabs module is mixed into PM::Screen, to use it, just prepare all the screens, declaring their title and icon, then init screens then call open\_tab\_bar(\*screens) in the main screen

## Styling the screen using teacup

Don't forget to set height and width attributes in the Teacup::Stylesheet, I forget to set it and get nothing to display in the screen.

## Fix the UITableView Hiding Under Scroll Bar in iOS7

to fix this, just use the navigation controller together.
