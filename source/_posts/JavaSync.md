---
title: Java Synchronization
date: 2022-04-12 22:57:00
categories:
    - Java
tags:
    - Language basic
---

## synchronized

### Unlocked

There are two bit indicates lock status, *lock* initiated with 01 and *bias lock* initiated with 0. Now, every thread can access this object without competition.

### Bias Lock

After the `synchronized` block was compiled, a `monitorenter` and `moniterexit` opcode will be inserted at the begainning and ending repectivly.

When thread A try to execute `monitorenter`, it try to update the **bias lock** to 1 by CAS and record it's id in MarkWord. Afert this, the same thread can enter directly as long as the MarkWord not changed.

### Lightwright Lock

When thread B try to gain the lock, it will trigger the *bias lock revocation*. The revocation phase will STW, and check the state of thread A, if it's exited the sync block, the lock will be recoveryed to unlocked state. Else the lock will be upgraded to **lightweight lock**.

Upgrade process:
1. Thread B create a **Lock Record** in it's stack.
2. Thread B use CAS to update the object MarkWord to the LockRecord in it's stack.
3. If CAS successful, thread B gained the lightweight lock, *lock* bit will be set to 00.
4. Else, thread B will spin and with.

### Heavyweight Lock

If the thread spin has failed too much times or it cost too much cpu resources, the lock will upgrade to **heavyweight lock**.

Upgrade process:
1. JVM create a ObjectMonitor object in heap.
2. Object Markword will be updated to a pointer pointing to that ObjectMonitor, *lock* bit will be set to 10.
3. Put all blocking threads to the **EntryList**, and wait for os dispath. This will trigger a switch from user mode to kernel mode, with maximum expenditure.

### Whey bias lock is deprecated?

The Bias lock is deprecated in JDK15 and removed in JDK18. In morden app, competition is always fierce, the expensive STW of bias lock revocation become unacceptable.

Other JVM optimization tech has reduce the benifits of bias lock:
1. Lock Elimination. 

