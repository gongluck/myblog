---
title: "搭建WSL的ssh服务"
date: 2020-10-17T16:33:15+08:00
draft: false

featured_image: "img/linux_wh.jpg"
categories: linux
tags: [linux]
---

```shell
sudo apt-get remove --purge openssh-server
sudo apt-get install openssh-server
sudo rm /etc/ssh/ssh_config
sudo service ssh --full-restart
```