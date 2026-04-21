---
title: IntelliJ Platform Plugin Dev Guide
date: 2026-04-14 14:59:00
categories:
    - IDE Plugin/Ext
tags:
    - IntelliJ Plugin
    - IntelliJ Platform
---

This article is about some IntelliJ plugin development experiences.

A plugin probject is a Gradle project, [IntelliJ](https://www.jetbrains.com/idea/) has provided tools to utilize the develop (but it's not absolutely necessary). Normally a plugin works for all IDEs based on the [IntelliJ Platform](https://www.jetbrains.com/opensource/intellij-platform), you can also specify which are supported in the [Marketplace](https://plugins.jetbrains.com/author/me) later.

A plugin probject is a Gradle project, IntelliJ has provided tools to utilize the develop (but it's not absolutely necessary).

Before you start, note that the [official forum](https://platform.jetbrains.com/) and [youtrack](https://youtrack.jetbrains.com/) can provide great helps for some tricky problems encountered in the developing.

