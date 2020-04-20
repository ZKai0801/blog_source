---
title: ssh快连+免密
date: 2019-06-10
tag: ["IT"]
---

由于工作需要，我往往会在多台服务器上反复切换，将文件从一个服务器传到另一个服务器上，因此会非常频繁的用到 `scp ssh`  这些命令，下面这两个功能能很大程度的省去其中的一些麻烦

## 简化SSH

首先在家目录下建一个`~/.ssh` 目录，然后编辑一个名为 `config` 的文件。

在这个文件中输入想要简化的服务器，例：

```bash
Host server1
  Hostname 123.456.78.910
  User kai
  ForwardX11 yes
```

并将此文件的权限给改成：

```bash
chmod 700 config
```

就大功告成了

之后想要链接那台服务器，只需要：

```
ssh server1
```

就okay了

## SSH 免密

免密的过程也十分简单，两步就能完成：

1. 先生成一对 public/private key

   ```bash
   ssh-keygen -t rsa
   ```

2. 把其中的public key 给放到另外那台服务器上

   ```bash
   ssh-copy-id -i ~/.ssh/id_rsa.pub remote_host
   ```

   