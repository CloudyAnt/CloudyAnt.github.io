---
title: Java Exception Handling
date: 2022-04-10 21:36:00
categories:
    - Java
tags:
    - Language basic
---

## OOM

1. Analyze the heap to find the most instantiated object type.
2. Follow the reference chain, find the the one cause the problem.

### How to analyze heap?

**Histogram**:
use `jmap -histo:live 86737 | head -n 20` to print the top memory consumer

**Dump & Open in analying tool**:
1. Use `jmap -dump:live,format=b,file=heap.bin` to dump live objects in binary format to file heap.bin
2. Open it in analying tool like Visual VM or MAT

**Visual VM**:
Connect the process in Visual VM, click 'Heap Dump', there will be a report generated

### OOM types

The possible OOM types are:
1. Java heap space. Heap space not enough, possibly because of memory leak or big objects. 
2. Metaspace. Normally due to there are too much dynamic classes.
3. GC overhead limit exceeded. Frequent gc with little space being squeezed out. Maybe it's due to memory leak, or overall heap size is too small.
4. Direct buffer memory. Buffer too small or buffer usage has no proper release. Always happens in nio network programming. Use `-XX:MaxDirectMemorySize` to adjust the size.

