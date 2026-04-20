---
title: TCP Connection Overview
date: 2026-04-20 20:35:00
tags:
    - HTTP
    - HTTP/2
    - TCP
categories:
    - Network
---

An overview of what a TCP connection is and why HTTP/2 multiplexing works.

## TCP connection essence

On the server side, a TCP connection is represented by a socket plus kernel state (often implemented as data structures in the OS network stack).

That state includes items such as:
1. Source and destination IP addresses, ports, and protocol
2. Sequence numbers
3. Acknowledgment numbers
4. Sliding-window information
5. Current TCP state (for example, `ESTABLISHED`, `FIN_WAIT_1`)

Since socket is just a file, an idle connection usually does not consume CPU time continuously. If a persistent connection is idle, the main costs are memory and file descriptor (FD) usage, not a dedicated thread running all the time.

"Close connection" means releasing the kernel resources for that socket and closing its FD. In normal TCP termination, this happens after the TCP four-way handshake completes.

This also explains how `Connection: keep-alive` works in HTTP/1.1: the client requests that the connection remain open for reuse. To avoid accumulating stale connections, servers still close idle connections after a timeout (Keep-Alive timeout) or after handling a configured maximum number of requests.

## HTTP/2 Multiplexing

**Multiplexing** is a core feature of HTTP/2. It allows multiple request/response exchanges to be active at the same time over one TCP connection.

As mentioned earlier, one TCP connection has one sequence space and one TCP state machine. So how can multiple HTTP messages run concurrently? The connection itself is still one TCP connection; multiplexing is implemented at the HTTP/2 framing layer.

HTTP/2 introduces **Stream** and **Frame**:
1. A **Stream** is a logical bidirectional channel for one request/response exchange and is identified by a **Stream ID**.
2. A **Frame** is the smallest protocol unit. Headers and body data are split into frames, and each frame carries its Stream ID.

Frames from different streams can be interleaved on the wire. The receiver uses Stream IDs to reconstruct each stream correctly.

Because of this model, HTTP/2 is naturally used with persistent connections.

From TCP's perspective, it is still ordinary TCP byte-stream transport.

### Weakness

The major weakness is *TCP Head-of-Line (HOL) blocking*: if one TCP packet is lost, later bytes in the same TCP stream cannot be delivered to the application until retransmission succeeds, even if those bytes belong to other HTTP/2 streams. **HTTP/3** addresses this at the transport layer by using QUIC over UDP.

### Header Overhead

If each frame carries Stream ID information, does that create extra overhead? It does add some framing overhead, but HTTP/2 reduces header cost with `HPACK` compression:
1. Static table: common header names and values can be represented by indexes
2. Dynamic table: recently seen headers can be reused by index (for example, repeated `user-agent` or `cookie` fields)

Overall, HTTP/2 multiplexing is more complex than HTTP/1.1, but in many real scenarios it improves latency and connection efficiency.