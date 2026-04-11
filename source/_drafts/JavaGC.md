---
title: Java GC
date: 2026-04-10 10:30:00
categories:
    - java
tags:
    - language basic
---

Java GC tech explanations.

## Concepts

### Throughput

The formula `(Total runnning time - GC time) / Totoal running time` to calculate throughput is perfect for Serial and Parallel. 

In a series of garbage collectors such as CMS and G1, there is a parallel *marking* phase, that makes things become complicate. In narrow sense, *CPU Cycle* should be invovled to do the calculation: `(Total CPU Cycles - GC CPU Cycles) / Totoal CPU Cycles`. But people normally only care about the realistic perspective, so a precise formula maybe `(Total running time - STW Time - Equivalent GC time in parallel phase) / Totoal running time`. Monitor the business performance change in the parallel phase to estimate the equivalent GC time.

Ways to increase throughput:
1. Biger heap. With a big heap, gc frequency will be decreased, and gc will be more concentrated.
2. Use a better gc algorithm.

Notice that concurrency may not increase the throughput, since there may not have multiple cores and task split will cost extra time. Then according to Amdahl's Law, acceleration times is limited.

### Pause time / Latency

Once SWT time.

### Footprint

Max used memory.

## Root Tracings

**GC Roots** Typically are:
1. Variables in thread stack
1. Static/Constant Variables

GC trace the reference graph from the GC Roots, it a object can not be reached, then it's eiligble for collection.

## Tri-color Marking

### Process

The object references is a graph, collector scan from **GC Roots** all the way down to the leafs.

In the very beginning, all objects are white. If an object is still white until the end, it will be collected.

If an object was visisted, it will firstly be painted to gray, then collector continue to scan it's field objects.

After all it's fields were scanned, it then be painted to black, which also means it will not be scanned again in this time.

The above content is very ideal, but it actually happens concurrenty to user threads, this might cause some problems.

### The Floating Garbage Problem

In the marking process, some marked objects might become grabage. They will be collected in current gc, but will be in next gc. So in most cases, this problem is accetable.

### The Lost Object Problem

When these two conditions happends **at the same time**, **Lost Object** problem will happen:
1. Assign a unreachable white object to the black object
2. Disconnect that white object from all gray objects

The result is that a living object might be wrongly cleaned. The core sulution is to record the two behaviors using **Write Barrier**:

**Incremental Update** breaks the condition 1.
When a black object refs to a white object, record it using write barrier. 
After marking finished, paint the recorded objects to gray and re-scan. 
Used in CMS.

**Snapshot At The Beginning, SATB** breaks the conditon 2.
When a gray object trys to remove the ref to a white object, put it to SATB queue using write barrier.
After marking finished, STW, for all objects in the SATB queue, paint all the objects in it's ref chain to black.
This surely might produce floating grabage, but it has higher efficiency.
Used in G1.
 
The record & re-paint works in both solutions are called **Remark**. 

## Collectors

### CMS

Concurrent Mark-Sweep. It will produce many memory fragments. 
Normally it won't compat space, but if old Gen space insurficient or no big enough sequential space for largee object, a full GC will be triggered.

Occations to STW:
1. Inital mark, find the root refs
2. Remark
3. Full GC 

### G1

Garbage First. It split memory space to equal-size regions, all regisons can be act as Eden, Survior or Old.
Globally it was based on Mark-Sweep, but from region it was based on Copying. 
In gc it move all objects from one region to another, so there is no memory fragment problems.

Most time there are only Minor GC and **Mixed GC** in G1. If Mixed GC cannot make enough space before out of memory, a Full GC will be triggered.

#### Mixed GC

When the Old gen space exceeded the threshold (Controlled by `-XX:InitiatingHeapOccupancyPercent`, default 45%), it will start a concurrent marking.
After marking, G1 knows which region has highest memory usage. Then it start serveral Mixed GC.
A mixed gc collect garbage from all Eden and Survior regison also the most worthwhile Old regions.

G1 won't collect all marked Old regions in one Mixed GC, but instead devide them to multiple times (Controlled by `-XX:G1MixedGCCountTarget`, default 8),
so to make user the pause not more than a specific time (Controlled by `-XX:MaxGCPauseMillis`)

### Humongous Object

Object sometime can be huge. In G1, if an object size exceeded 50% capacity of a region, it was then called a humongous object.

This kind of object was be created in **Humongous Region** instead of Eden, it's similar to a Old regison. If the object size exceeded the size of region, it will then be placed in a **Continuous Humongous Region**.

Objects in these region won't be frequently collected, can only be collection by Mixed GC and Full GC.

### ZGC


