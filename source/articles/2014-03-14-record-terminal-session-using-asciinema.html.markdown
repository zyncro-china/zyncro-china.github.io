---
title: Record terminal session using asciinema
date: 2014-03-14 09:09 UTC
tags: screencast, terminal
---

## My IDE is based on the Terminal 

## Install asciinema

```shell

sudo pip install --upgrade asciinema
```
## Usage
### Authenticate yourself
In order to be able to delete or edit your terminal screencast, it require to
login first. Use the below command and follow the instructions. 

```shell

asciinema auth
```

### Start to record the session

```shell

asciinema
```

### To embed a terminal screencast in a page
We can embed a screencast using javascript as below:

<script type="text/javascript" src="https://asciinema.org/a/4348.js"
id="asciicast-4348" async></script>
