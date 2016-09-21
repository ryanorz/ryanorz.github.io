---
layout:     post
title:      "Something interesting in Linux multi-process"
subtitle:   "many same signals are merged to one signal"
date:       2016-05-23
author:     "Ryan"
header-img: "img/post-bg-2015.jpg"
tags:
    - C/C++
    - linux
    - Programming
---

# signal merge

When same signals are sent at the same time, the signal handle function will only be called one time. So the `wait` need to be call many times.

# Example

```c
void wait_child(__atttribute__((unused))int sig)
{
    int stat;
    int pid;
    while ((pid = waitpid(-1, &stat, WNOHANG)) > 0) {
        //TODO do something
    }
}

struct sigaction act;
act.sa_flags = 0;
act.sa_handler = wait_child;
sigemptyset(&act.sa_mask);
sigaction(SIGCHLD, &act, 0);
```
