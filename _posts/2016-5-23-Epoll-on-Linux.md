---
layout:     post
title:      "Epoll on Linux"
subtitle:   "Epoll: I/O event notification facility"
date:       2016-05-23
author:     "Ryan"
header-img: "img/post-bg-2015.jpg"
tags:
    - C/C++
    - linux
    - Programming
---

# Description

The  epoll API performs a similar task to poll(2): monitoring multiple file descriptors to see if I/O is possible on any of them.

# API

```c
#include <sys/epoll.h>

/**
 * @note:   need use close() to close fd.
 * @param:  size - since Linux 2.6.8 is ignored, but must be ignored than zero.
 * @flags:  flags - if 0, is the same as epoll_create. EPOLL_CLOEXEC
 * @return: a file descriptor
 */
int epoll_create(int size);
int epoll_create1(int flags);

/**
 * @param: op - EPOLL_CTL_ADD, EPOLL_CTL_MOD, EPOLL_CTL_DEL
 */
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);

int epoll_wait(int epfd, struct epoll_event *events,
               int maxevents, int timeout);
```

# Example with timer

```c
#include <sys/epoll.h>
#include <sys/timerfd.h>

int init_timer(int epfd)
{
    int timefd = timerfd_create(CLOCK_MONOTONIC, TFD_CLOEXEC | TFD_NONBLOCK);
    if (timefd == -1) {
        syslog(LOG_ERR, "timerfd_create: %m");
        return -1;
    }

    // bind timefd to epfd
    struct epoll_event event;
    event.events = EPOLLIN;
    event.data.fd = timefd;
    if (-1 == epoll_ctl(epfd, EPOLL_CTL_ADD, timefd, &event)) {
        syslog(LOG_ERR, "epoll_ctl: %m");
        close(timefd);
        return -1;
    }

    itimerspec new_timer;
    itimerspec old_timer;
    new_timer.it_interval.tv_sec = 2;
    new_timer.it_interval.tv_nsec = 0;
    new_timer.it_value.tv_sec = 2;
    new_timer.it_value.tv_nsec = 0;
    if (-1 == timerfd_settime(timefd, 0, &new_timer, &old_timer)) {
        syslog(LOG_ERR, "timerfd_settime: %m");
        close(timefd);
        return -1;
    }
    return timefd;
}

#define MAX_EVENTS 10
#define TIMEOUT_IFINITY -1

int epfd = epoll_create1(EPOLL_CLOEXEC);
if (epfd == -1) {
    syslog(LOG_ERR, "epoll_create1: %m");
    return -1;
}

int timefd = init_timer(epfd);
if (-1 == timefd) {
    close(epfd);
    return -1;
}

struct epoll_event event_pool[MAX_EVENTS];
int num_fd;
for (;;) {
    num_fd = epoll_wait(epfd, event_pool, MAX_EVENTS, TIMEOUT_IFINITY);
    if (num_fd == -1 && errno != EINTR && errno != ERESTART) {
        syslog(LOG_ERR, "epoll_wait: %m");
        exit(-1);
    }
    if (errno == EINTR || errno == ERESTART) {
        //@note system interrupt
        //TDDO do something
    }
    if (nfds == 0) {
        //@note interrputed by other signal, such as SIGCHLD
        //TODO do something
    }
    for (int i = 0; i < nfds; ++i) {
        if (event_pool[i].data.fd == timefd) {
            //@note accept timer event
            //TODO do something
        } else {
            //@note other event added by epoll_ctl EPOLL_CTL_ADD
            //TODO do something
        }
    }
}
```
