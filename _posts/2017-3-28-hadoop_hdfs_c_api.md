---
layout:     post
title:      "Hadoop HDFS C API 使用经验"
date:       2017-3-28
author:     "Ryan"
header-img: "img/post-bg-2017.jpg"
tags:
    - Hadoop
    - HDFS
---

# Sample Code

[hadoop sample code](http://hadoop.apache.org/docs/r2.7.3/hadoop-project-dist/hadoop-hdfs/LibHdfs.html)

## Hadoop HDFS C API　cmake

```cmake
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -Wall -O2")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_REENTRANT -D_GNU_SOURCE")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64")

find_package(JNI REQUIRED)

include_directories(
    ${JNI_INCLUDE_DIRS}
    ${CMAKE_SOURCE_DIR}/include # hdfs.h
)

add_executable(hadoop_cpp main.cpp)
target_link_libraries(hadoop_cpp
    ${CMAKE_SOURCE_DIR}/lib/native/libhdfs.so
    ${JAVA_JVM_LIBRARY}
)
```

# Environment Variables

Add ENV in `.bashrc`:

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=~/package/hadoop-2.7.3
export CLASSPATH=`${HADOOP_HOME}/bin/hadoop classpath --glob`
```
如果不设置 CLASSPATH, 会出现如下错误．
```
Environment variable CLASSPATH not set!
getJNIEnv: getGlobalJNIEnv failed
Environment variable CLASSPATH not set!
getJNIEnv: getGlobalJNIEnv failed
Failed to open /tmp/testfile.txt for writing!
```

`--glob` 必须加．否则会出现如下错误．

```
loadFileSystems error:
(unable to get stack trace for java.lang.NoClassDefFoundError exception: ExceptionUtils::getStackTrace error.)
hdfsBuilderConnect(forceNewInstance=0, nn=default, port=0, kerbTicketCachePath=(NULL), userName=(NULL)) error:
(unable to get stack trace for java.lang.NoClassDefFoundError exception: ExceptionUtils::getStackTrace error.)
Failed to connect to hdfs.
```
