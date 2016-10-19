---
layout:     post
title:      "Linux中的定时器"
date:       2016-10-19
author:     "Ryan"
header-img: "img/post-bg-2015.jpg"
tags:
    - C/C++
    - linux
    - Programming
---

# 1. `timer_create`

## 1.1 先看个例子

```c++
/**
 * 链接时需要 -lrt
 */
#include <time.h>
#include <signal.h>

void handle_alarm(__attribute__((unused))int sig)
{
	//TODO deal signal
}

void set_timer()
{
	// Step1: register signal action, 注册信号处理
	struct sigaction act;
	act.sa_flags = 0;
	act.sa_handler = handle_alarm;
	sigemptyset(&act.sa_mask);
	sigaction(SIGALRM, &act, 0);

	// Step2: 创建定时器
	timer_t timerid;
	if (-1 == timer_create(CLOCK_REALTIME, NULL, &timerid))
		log_exit("timer_create");

	// Step3: 设置时间，启动定时器
	int timeout = 2; // timeout 2 seconds
	struct itimerspec new_timer, old_timer;
	new_timer.it_interval.tv_sec  = auto_interval;
	new_timer.it_interval.tv_nsec = 0;
	new_timer.it_value.tv_sec     = first_trigger;
	new_timer.it_value.tv_nsec    = 0;
	if (-1 == timer_settime(conninfo.request.timerid, 0, &new_timer, &old_timer)) {
		timer_delete(timerid);
		log_exit("timer_settime");
	}
}
```

timer_create()在触发的时候默认会触发SIGALRM信号，当然，这些动作可以设置参数修改．
需要注意的是创建的timerid不会在创建的子进程中被继承．
具体函数用法看 MAN 手册会比较好．

## 1.2 `struct itimerspec`

* it_interval: 自动触发定时器间隔
* it_value：　第一次触发定时器的间隔
* tv_sec: 秒
* tv_nsec: 纳秒

# 2. `timerfd_create`

与timer_create用法类似，但是，是创建文件描述符，一般用epoll监听．
`These system calls are available on Linux since kernel 2.6.25.  Library support is provided by glibc since version 2.8.`
所以在做项目中发现CentOS 5.0中不支持．

```c++
#include <sys/timerfd.h>

int set_timer()
{
	// Step1: 创建定时器
	int timefd = timerfd_create(CLOCK_MONOTONIC, TFD_CLOEXEC | TFD_NONBLOCK);
	if (timefd == -1)
		log_exit("timerfd_create");

	// Step2: 设置时间，启动定时器
	struct itimerspec new_timer, old_timer;
	new_timer.it_interval.tv_sec = 2;
	new_timer.it_interval.tv_nsec = 0;
	new_timer.it_value.tv_sec = 2;
	new_timer.it_value.tv_nsec = 0;
	if (-1 == timerfd_settime(timefd, 0, &new_timer, &old_timer)) {
		close(timefd);
		log_exit("timerfd_settime");
	}
	return timefd;
}
```


































