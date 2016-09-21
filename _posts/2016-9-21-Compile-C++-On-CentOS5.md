---
layout:     post
title:      "Compile My C++ program on CentOS 5."
subtitle:   "What a terrible experience."
date:       2016-09-21
author:     "Ryan"
header-img: "img/post-bg-2015.jpg"
tags:
    - Programming
    - linux
    - C/C++
---

# Background

应工作需求, 要将我的代码编译到CentOS 5平台上. 实际当初的开发及运行环境是Ubuntu 16.04, 以及 CentOS 7.
但是现在要将东西搞到CentOS 5上, 想着这么多年Linux内核及各种库的变化就觉得这不是个好玩的事.但还是得搞不是.

# 第一个坑

我承认, 我用到的各种库特别的多, 因为我不想花时间造轮子, 而且轮子的质量也不一定高. 于是,刚一开始就发现这个天坑.
依赖的库太多了, 各种版本太低而导致不能用的库, 还有各种没有的库. 好吧, 这都得自己编.问题是库和依赖的程序的构建
环境也要自己编, 库的依赖库也要编译.差点连cmake和mercurial都自己编译一遍. 总共编译了几十个库, 都要吐了, 是不
是承受能力有点低.其实之前还编译过这些东西的mingw版本.感觉那个更恶心.

# C++11

终于可以编译我自己的代码了. 离成功只有一步之遥,剩下的就简单多了.
CentOS 5 的`gcc`还不支持`-std=c++0x`,`clang`甚至还没有, 幸亏有神器`boost`,免除了我多少痛苦.
`local_thread`没有, 咱用`boost`; `unordered_map`没有,咱用`boost`. boost大法好 (:D).
但是cstdint, cstdio不能include, 只能stdint.h和stdio.h ^-^~有点意思.
还有`std::to_string`和 `std::stoi` 不能用的问题(好吧,都是小问题).
`unique_ptr`也没有, 连`0b`表示二进制数也不行.

# Linux Kernel

剩下最后一个问题, 内核太旧,有些函数和有些宏还不能用.
`SOCK_NONBLOCK, SOCK_CLOEXEC`, `accept4()`, `EPOLL_CLOEXEC`, `epoll_create1`没有;
`CLONE_NEWNET`对于`clone()`这函数还没加进来;`be16toh(x)`这种宏也没有.


改掉之后终于编完了,整个过程花了好几天时间.
