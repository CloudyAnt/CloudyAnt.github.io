---
title: Java Tools
date: 2026-04-10 21:35:00
categories:
    - java
tags:
    - language basic
---

Description of common built-in tools are `jps`, `jstat`, `jmap`, `jinfo`, `jhat`, `jstack`, `jconsole`.
Check `--help` of each command or go to the oracle [offical docs](https://docs.oracle.com/en/java/javase/11/tools/index.html) for a comprehensive explanation.

## jps

List pid of current running java process. 

If a machine has `jstatd` running, you can check it remotely by `jps $targetIp`. But make sure the 1099 and data-transfer port are open:

```shell
# Assume that we are goint to use 1099 and 2026 port 
jstatd -J-Djava.rmi.server.hostname=<Public IP> -p 1099 -J-Dcom.sun.management.jmxremote.port=2026 &
```

## jstat

Monitor JVM statistics of multiple aspects.

### gcutils

Show the gc status. usage example: 

```shell
# Print gc info every 250ms, execute 7 times
# Both the interval and times to print are optional
jstat -gcutils $pid 250 7
```

> There is a **CCS** column in gctutils output, the means Compressed Class Space. 
> Most of the morden os are 64-bit, but the object pointers and class metadatas might not need 64-bit space, so they need such tech to be compressed.

## jcmd

Send diagnostic command requests to a running JVM. Use `jcmd $pid help` to show available commands for specifid process. For example:

```shell
# Show gc heap info
jcmd $pid GC.heap_info
```

> The **GC.heap_info** shows the *commited* and *reservered* size, the former is the actually required space, the later is the virtual space.

## jmap

Print details of a specific process. For example, you can dump the heap by:

```shell
# Dump live objects in binary format to file heap.bin
jamp -dump:live,format=b,file=heap.bin $pid

# Dump live objects in histogram to file histo.data
jmap -histo:live,file=/tmp/histo.data $pid
```

## jinfo

Show everything about a process, even change some of them, for example:

```shell
# Change the thread dump file path
jinfo -flag HeapDumpPath=/var/log/dump $pid
```

Use `jcmd $pid VM.flags -all` to show all possible flags of that process.

## jstack

Print thread stack info, that contains thread status and resources usage info.

## jconsole

A GUI statistics monitor

## jhsdb

Stands for java hostspot debugger.

## javap

Disassemble a class file so user can inspect the bytecode and internal structure.