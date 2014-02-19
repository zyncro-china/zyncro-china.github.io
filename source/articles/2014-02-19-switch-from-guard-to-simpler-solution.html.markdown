---
title: switch from guard to simpler solution
date: 2014-02-19 01:43 UTC
tags:
---
# The weird problem when using guard
I have recently encounter a problem: when using guard to watch file changes,
and run tasks automatically, they run multiple times, or remove the result.
[Some similiar issues on github](https://github.com/netzpirat/guard-coffeescript/issues/28)

# Fix the problem or avoid it

It seems the problem may be caused by editor, OS, or some gems, I have tried
some time, but can't find a solution. All I need is just run some task
automatically when file changes. So I try this rubygem:
[when-files-change](https://github.com/adamsanderson/when-files-change) It
should be simpler than Guard, and I don't need to write config file, just type:

```shell

when-files-change -- rake test
when-files-change --ignore 'build' -- make
```

That is all. I can leave the task run by itself without manually invoke them.

# Guard come back to work
When I create a new project later, some guard tasks run normally. Maybe my
previous problem is caused by some rubygem dependencies.

# Other solutions that run tasks from vim by key mappings

## use tmux to run tasks asynchronously

```shell

tmux new-session -n development
```
tmux can send keys to another pane like

```shell

tmux send-keys -t development.1 "echo test" C-m
tmux send-keys -t development.1 "rake" C-m
```

## use fifo or pipe to run tasks:

```shell

mkfifo test-commands

in one screen: cat test-commands
in another screen: echo test-commands
```
So now we can use the fifo/pipe to send the command over.

```shell
sh -c "$(cat test-commands)"
```

to make it run forever:

```shell
while true; do sh -c "$(cat test-commands)"; done
```

use vim mapping to run the command on current file, silently through the shortcut, and redraw:

```shell
remap <leader>E :w\|:silent !echo "bundle exec ruby %" > test-commands<cr>\|:redraw!<cr>
```

write a run-test.sh to avoid create the fifo manually, because fifo can't check into git repository:

```shell

if [ ! -p test-commands ]
then
  mkfifo test-commands
fi

while true; do
  sh -c "$(cat test-commands)"
done

```
