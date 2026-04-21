---
title: Java GC
date: 2022-04-10 10:30:00
categories:
    - Java
tags:
    - Language basic
---

Java GC fundamentals and modern collector internals.

## JVM Heap Layout

The JVM heap is divided into two main areas:

- **Young Generation**: Where newly allocated objects live. Split into:
  - **Eden**: Most objects are allocated here.
  - **Survivor S0 / S1**: Objects that survive a Young GC are copied between these two spaces. Each surviving object has an *age counter* incremented per GC cycle.
- **Old Generation (Tenured)**: Long-lived objects promoted from Young Gen after exceeding the age threshold (`-XX:MaxTenuringThreshold`, default 15).
- **Metaspace** (off-heap): Stores class metadata. Replaced PermGen since Java 8.

## Object Lifecycle

1. New objects are allocated in **Eden**.
2. When Eden fills up, a **Minor GC** (Young GC) is triggered. Live objects are copied to a Survivor space and their age increments.
3. Objects exceeding the age threshold are **promoted** to Old Gen.
4. When Old Gen fills up (or under certain collector-specific conditions), a **Major GC** or **Full GC** is triggered, collecting the whole heap.

Cross-generational references (e.g. an Old Gen object referencing a Young Gen object) are tracked via a **Card Table** / **Remembered Set**, so Minor GC does not need to scan all of Old Gen to find roots.

## GC Metrics

### Throughput

`(Total running time - GC time) / Total running time`

This simple formula is accurate for stop-the-world collectors like Serial and Parallel. For concurrent collectors such as CMS and G1, a parallel marking phase runs alongside application threads, which complicates the calculation. A more precise measure in CPU terms is `(Total CPU Cycles - GC CPU Cycles) / Total CPU Cycles`. In practice, people estimate by measuring business performance degradation during the concurrent phase.

Ways to increase throughput:
1. **Larger heap**: Lower GC frequency, though each GC event may take longer.
2. **Better GC algorithm**: e.g., moving from Serial to Parallel or G1.

Note that adding concurrency does not always increase throughput — thread-coordination overhead and Amdahl's Law impose limits, especially on machines with few cores.

### Latency (Pause Time)

Also known as **STW (Stop-The-World) time** — the duration during which all application threads are suspended for GC work. This directly impacts application responsiveness.

### Footprint

Total memory consumed by the JVM process, including heap and off-heap areas (Metaspace, JIT code cache, etc.). Relevant in memory-constrained environments.

## GC Root Tracing

**GC Roots** are the starting points for reachability analysis. Typically:
1. Local variables and operand stack entries across all thread stacks
2. Static fields and constants (class-level references)
3. JNI references

The GC traces the object reference graph from GC Roots. Any object that cannot be reached is eligible for collection.

## Tri-color Marking

### Process

All objects start as **white**. Objects still white at the end of marking are collected.

When an object is first visited, it is painted **gray** — visited, but its fields have not yet been scanned.

Once all of an object's fields are scanned, it is painted **black** — fully processed and will not be scanned again in this cycle.

Because marking runs concurrently with application threads, two correctness problems can arise.

### The Floating Garbage Problem

During marking, some already-marked (black) objects may become unreachable due to reference changes by the application. These "floating garbage" objects are missed in the current GC cycle and collected in the next one. This is generally acceptable.

### The Lost Object Problem

A live object can be incorrectly collected when **both** conditions happen concurrently:
1. A black object gains a reference to a white object.
2. All gray objects that previously referenced that white object drop those references.

The white object is never painted gray/black and gets wrongly collected. The fix is to use a **Write Barrier** to record one of the two mutations:

**Incremental Update** (breaks condition 1 — used by CMS):
When a black object acquires a reference to a white object, record it.
After marking finishes, re-paint the recorded objects gray and re-scan them (Remark STW).

**Snapshot At The Beginning (SATB)** (breaks condition 2 — used by G1):
When a gray object is about to drop a reference to a white object, push that white object onto the SATB queue.
After marking finishes, during a Remark STW, process the SATB queue and paint those objects' reference chains black.
This may produce more floating garbage but has lower re-scan overhead.

The re-scan step in both approaches is called **Remark**.

## Collectors

### Serial GC

Single-threaded; stops all application threads during collection. Minimal overhead and simple implementation. Suitable for single-core environments or very small heaps.

- Young Gen: **Serial** (mark-copy)
- Old Gen: **Serial Old** (mark-sweep-compact)

### Parallel GC

Multi-threaded version of Serial. Uses multiple threads for GC work to maximize throughput. Was the default collector before Java 9.

- Young Gen: **Parallel Scavenge**
- Old Gen: **Parallel Old**

### CMS

**Concurrent Mark-Sweep**. Minimizes pause time by doing most marking work concurrently with application threads. Does not compact the heap, which causes memory fragmentation over time.

Full GC (single-threaded, compacting) may be triggered if:
- Old Gen space is insufficient for promotion.
- No sufficiently large contiguous free space exists for a large object.

Phases:
1. **Initial Mark** — Find root references. (STW)
2. **Concurrent Mark** — Traverse the reference graph. (Concurrent)
3. **Remark** — Re-scan mutated references via Incremental Update. (STW)
4. **Concurrent Sweep** — Reclaim dead objects. (Concurrent)

> CMS was deprecated in Java 9 and removed in Java 14.

### G1

**Garbage First**. Splits the heap into equal-size **Regions** (~1–32 MB each). Each region can act as Eden, Survivor, or Old Gen at different times.

Globally it is **Mark-Sweep** based; per-region it is **Copying** based — objects are evacuated to new regions, eliminating memory fragmentation.

Normal operation involves **Minor GC** (Young-only) and **Mixed GC**. A **Full GC** is triggered only if G1 cannot reclaim space fast enough.

#### Mixed GC

Triggered when Old Gen occupancy exceeds a threshold (`-XX:InitiatingHeapOccupancyPercent`, default 45%):
1. Concurrent marking runs to determine region liveness.
2. G1 selects the Old Gen regions with the most garbage (highest collection value).
3. Mixed GC collects all Young regions plus a subset of those Old Gen regions.

G1 spreads Old Gen collection across multiple Mixed GC cycles (`-XX:G1MixedGCCountTarget`, default 8) to stay within the pause target (`-XX:MaxGCPauseMillis`).

#### Humongous Objects

If an object's size exceeds 50% of a region's capacity, it is a **Humongous Object** and is allocated directly into one or more consecutive **Humongous Regions** (treated like Old Gen). Humongous objects are only collected during Mixed GC or Full GC.

### ZGC

The [ZGC](https://docs.oracle.com/en/java/javase/25/gctuning/z-garbage-collector.html) performs almost all work concurrently with application threads. Its goal is to keep STW pauses under a few milliseconds regardless of heap size.

#### Reference Coloring

ZGC stores GC state in **bits of the reference pointer itself** rather than in the object's mark word or a separate bitmap. This technique is called **Reference Coloring**, and it only works with 64-bit references.

In a 64-bit pointer, the lower 42 bits address the object; 4 middle bits encode the reference state (`marked0`, `marked1`, `remapped`, `finalizable`). Because mutating the state bits changes the virtual address, ZGC uses **Multi-Mapping** to ensure all those different virtual addresses map to the same physical memory page.

#### ZPage

Similar to G1's Region, but ZPages have three fixed size classes:

| Type   | Size     | Object size range | Notes |
|--------|----------|-------------------|-------|
| Small  | 2 MB     | < 256 KB          | Most numerous; relocated most frequently |
| Medium | 32 MB    | 256 KB – 4 MB     | Relatively stable |
| Large  | N × 2 MB | > 4 MB            | Dynamic size; one object per page |

Each ZPage has a **Live Map** bitmap that tracks object liveness by object offset within the page.

#### Marking Phase

1. **STW**: Find and mark root references.
2. **Concurrent**: Traverse the reference graph. **Load Barriers** mark any reference that is loaded but found to be unmarked.
3. **STW**: Handle edge cases (references modified during step 2).
4. Each traversed reference sets `marked0` or `marked1` in the colored pointer, and the corresponding bit in the Live Map.

#### Relocation Phase

**Preparing**:
1. Scan all ZPages; select those with the most garbage (fewest surviving objects) into a **Relocation Set**.
2. Reserve free ZPages as relocation targets.

**Execution**:
1. **STW**: Relocate all objects referenced by roots in the Relocation Set; update those root references.
2. **Concurrent**: Relocate remaining objects. Old-to-new address mappings are stored in a **Forwarding Table** per ZPage.
3. Remaining stale references are fixed lazily during the next marking phase or by Load Barriers (see below).

#### Remapping

When the application loads a reference pointing to an object that has been relocated, the **Load Barrier** intercepts it:
1. If the `remapped` bit is set, return the reference as-is.
2. Otherwise, look up the Forwarding Table, update the reference to the new address, set the `remapped` bit, and return it.

This ensures all stale references are healed lazily without a separate STW fixup pass.

## Collector Comparison

| Collector | Primary Goal   | Heap Size | Available Since | Notes |
|-----------|---------------|-----------|-----------------|-------|
| Serial    | Simple        | Small     | Java 1          | Single-threaded STW |
| Parallel  | Throughput    | Medium    | Java 1.4        | Multi-threaded STW; default pre-Java 9 |
| CMS       | Low latency   | Medium    | Java 1.4        | Fragmentation; removed in Java 14 |
| G1        | Balanced      | Large     | Java 7          | Default since Java 9 |
| ZGC       | Sub-ms pauses | Any       | Java 15 (prod)  | Best for large heaps or strict latency |
