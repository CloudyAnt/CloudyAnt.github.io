---
title: Java Class & Object
date: 2022-04-11 16:11:00
categories:
    - java
tags:
    - language basic
---

## Object

### lifecycle

1. load related class
2. allocate memory, and do calcluations in costructor
3. live then became invisible or unreachable
4. be collected by gc
5. finalize and deallocate

### Format in memory

**Headers**:
1. Mark word. Store runtime data, normally 8 bytes.
2. Klass pointer. Ref to the class meta data in Method Area. If `-XX:+UseCompressedClassPointers` enabled, it will be compressed from 8 bytes to 4 bytes.
3. Array length. Only exists when it's array. Used to store size of array, 4 bytes.

**Instance Data**:
The business data in here. The order of them may be reordered by JVM, and ther super class fields appears before those in sub class.

**Padding**:
The Hostspot internal allocation strategy requires that the starting address of an object must be an integer multiple of 8 bytes, so JVM may append some meaningless chars.

## Class

### Delegation Mode

```mermaid
flowchart LR
cus[Custom CL] --> app[App/System CL] --> ext[Ext CL] --> boot[Bootstrap CL]

boot -.-> ext -.-> app -.-> cus
```

Each class loader delegate the loading to it's parent, if their parent cannot load, then it will try to load. This is called the **Delegation Mode**.

`Class.forName` use the classloader of current object, or the default app/system classloader.

By specifying a custom classloader, you may break the mode, for example, download a class from network.
