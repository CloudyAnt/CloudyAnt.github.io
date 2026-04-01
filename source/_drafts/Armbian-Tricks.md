---
title: Armbian-Tricks
date: 2026-04-01 17:46:00
tags:
    - OS
    - Armbian
    - Linux
categories:
    - Armbian
---

## Problems

**Correct xterm-ghostty keymap**

If connected by ssh in `ghostty`, you may encounter a problem that thenbackspace key performs wrong action instead of delete char.
That because armbian may not recognize the xterm-ghostty `TERM` sign. Register/Fix it by:

```shell
infocmp -x | ssh user@host "tic -x -"
```
