---
title: Netty
date: 2026-04-13 09:56:00
categories:
    - Netty
tags:
    - Netty
    - Network Communication
---


## Backpresure

There is a bufer called **ChannelOutboundBuffer** in nettry, if it's data size exceeded **High Watermark**(default 64K), Netty will set the **isWritable** to false. Then if the data size if lower than **Low Watermark**(default 32K), Netty will reset is to true.

Be ware that the `isWrtiable = false` is not a mandatory limit, you can still write into the buffer, but it's strongly **not recommanded**.  You'd better check the `channel.isWritable()` before write.

## Optimization for massive connections

1. Buffering. Store the excess data in memory. Suitable for senarios that with surficient memory, or data losing unacceptable.

2. Dropping. 

