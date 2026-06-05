---
title: Gerrit Tips
date: 2026-06-05 15:04:00
tags:
    - Git 
    - Gerrit 
categories:
    - Gerrit 
---

## Problems

**No matching key exchange method found**

it looks like this:

```txt
Unable to negotiate with 127.0.0.1 port 12345: no matching key exchange method found. Their offer: diffie-hellman-group14-sha1,diffie-hellman-group1-sha1

fatal: Could not read from remote repository. 
```

This because Gerrit use old version of algorithm, but you local git/ssh version is relatively new. The new version of SSH client disabled the low security key exchange method. You have to enable them at `~/.ssh/config`:

```txt
Host youcompany.gerrit.host 
    KexAlgorithms +diffie-hellman-group14-sha1,diffie-hellman-group1-sha1
    HostKeyAlgorithms +ssh-rsa
```

**fatal: Could not read from remote repository.**

If you confirmed that you added the correct SSH key to the Gerrit, then it must due to the Gerrit server cannot find the matched key，expecially you have multiple keys.

To solve it, add this lines:

```txt
Host mycompany.x3322.net
    IdentityFile ~/.ssh/id_rsa
    KexAlgorithms +diffie-hellman-group14-sha1,diffie-hellman-group1-sha1
    HostKeyAlgorithms +ssh-rsa
    PubkeyAcceptedKeyTypes +ssh-rsa
```

