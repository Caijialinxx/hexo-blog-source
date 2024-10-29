---
title: 命令行操作时遇到的一些报错解决方案（持续更新...）
date: 2023-03-24 19:30:18
tags: [troubleshooting, 工作]
---

## Github - SSH
官网文档：https://docs.github.com/en/authentication/troubleshooting-ssh

### REMOTE HOST IDENTIFICATION HAS CHANGED!
> Error: @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
> @    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
> @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
> IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
> Someone could be eavesdropping on you right now (man-in-the-middle attack)!
> It is also possible that a host key has just been changed.
> The fingerprint for the RSA key sent by the remote host is
> SHA256:uNiVztksCsDhcc0u9e8BujQXVUpKZIDTMczCvj3tD2s.
> Please contact your system administrator.
> Add correct host key in /Users/caijialinxx/.ssh/known_hosts to get rid of this message.
> Offending RSA key in /Users/caijialinxx/.ssh/known_hosts:1
> Host key for github.com has changed and you have requested strict checking.
> Host key verification failed.
> fatal: Could not read from remote repository.
> 
> Please make sure you have the correct access rights
> and the repository exists.

Github 远程主机地址变更，导致 SSH 链接失败。解决方法：
- 在 `~/.ssh/known_hosts` 删除对应IP的host记录
- 或者删除整个 `~/.ssh/known_hosts` 文件



## 其他

### xcrun: error: invalid active developer path
> xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun

更新完 macOS 版本之后，使用终端出现这个报错，是因为 xcode 在更新后可能会需要重新安装。执行下行命令重装 xcode 即可：
```bash
$ xcode-select --install
```

