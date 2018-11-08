---
layout: post
title: "Jekyll 安装权限问题 ERROR: While executing gem ... (Gem::FilePermissionError)"
date: 2018-03-07
desc: "Jekyll 安装权限问题 ERROR: While executing gem ... (Gem::FilePermissionError)"
keywords: "mancoxu, Jekyll"
categories: [Other]
tags: [Jekyll]
icon: icon-html
---

OS X El Capitan 新特性(System Integrity Protection or SIP)中加强了权限，但是可以对这里进行操作 /usr/local/bin

可以尝试使用以下指令进行 jekyll 的安装（亲测可行，安装完毕后 terminal 中输入 jekyll 即可看到是否生效）：

```
sudo gem install -n /usr/local/bin/ jekyll
```

这条指令告诉 gem，把 jekyll 安装到不受 SIP 保护的文件夹，而不是安装到默认/Library/Ruby/Gems 下的目录中。
