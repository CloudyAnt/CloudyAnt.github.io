---
title: OpenClaw Tricks
date: 2026-03-29 17:15:35
categories:
  - AI
  - OpenClaw
tags:
  - OpenClaw
  - AI
  - Agent
  - Tools
---


For basic knowledges and usages about openclaw just check its [github page](https://github.com/openclaw/openclaw)

## Empowerment

To make it really open **claw**, it's necessary to grant it the access to `exec` command, otherwise it's just a **chatter**. 
To do so, edit the **openclaw.json**:

```json
 "tools": {
    "profile": "full",
    "allow": ["*"],
    "exec": {
      "host": "gateway",
      "security": "full",
      "ask": "off"
    }
  }
``` 

## Skills

Skills can be found at [clawhub](https://clawhub.ai/), [skills.sh](https://skills.sh/), etc. They have guides about how to install.

## Model Config

You can edit the **openclaw.json** directly to add models, but it's more suggested to use the `openclaw onboard` command instead, even though it feels a litter semantically incorrect.

## Channel Config

Add any social media channel will give you the ability to dictate that lobster remotely.

**QQ Bot**

Get an *app id* and *app secret* first at [here](https://q.qq.com/qqbot/openclaw/index.html), then:

```shell
# Install the plugin
openclaw plugins install @tencent-connect/openclaw-qqbot@latest
# Bind the robot
openclaw channels add --channel qqbot --token "$appId:$appSecret"
# Restart service
openclaw gateway restart
```

