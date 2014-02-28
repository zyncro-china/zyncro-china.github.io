---
title: Compile vim source with python and ruby support
date: 2014-02-28 05:52 UTC
tags:
---
vim,ruby,python

### Compile the vim after upgrade ruby to 2.1.1

After I upgrade to ruby 2.1.1, I found the vim can't work, so I get the source of vim and compile it again.

It is weird that the src/Makefile can't take effect, I have to manually input the configure parameters:

```shell

 ./configure --with-features=HUGE \
--enable-multibyte=yes \
--enable-rubyinterp=yes \
    --with-ruby-command=/home/david/.rvm/rubies/ruby-2.1.1/bin/ruby \
    --enable-gui=gnome2 \
    --with-x \
    --enable-pythoninterp=yes \
--with-python-config-dir=/usr/lib/python2.7/config/


make

sudo make install

```

For the Command-T plugin:

```shell

cd ruby
ruby extconf.rb
make
```

Done!

