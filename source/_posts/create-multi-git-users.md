---
title: 创建多个Git用户、推送多个远程仓库
date: 2020-03-06 17:13:44
tags: [工作, Notes]
keywords: git, multi, config, ssh
---

## 创建多个Git用户
1. 生成多个ssh-key，并将对应的publicKey添加到git管理仓库中
    ```bash
    $ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    ```
2. 配置~/.ssh/config
    ```bash
    # The git info for company
    # gitlab user(your_email1@example1.com)
    Host git@x.x.x.x
      HostName x.x.x.x
      PasswordAuthentication no
      Port xxxx
      User your_email1
      IdentityFile ~/.ssh/example1/id_rsa

    # The git info for my github
    # Default github user(your_email2@example2.com)
    Host git@github.com
      HostName github.com
      User your_email2
      IdentityFile ~/.ssh/example2/id_rsa
    ```
3. 复制完整的 SSH 公钥并在对应的远程仓库中添加 SSH key 配置。
4. 验证 SSH 连接
    ```bash
    $ ssh -T [Host]
    # 如 ssh -T git@github.com
    ```


## 推送项目到不同仓库地址
1. 进入项目，添加远程仓库B地址
    ```bash
    $ git remote add 远程仓库B的别名（随意） 远程仓库B的地址
    ```

2. 在A项目中将commit提交到远程仓库B
    ```bash
    $ git push 远程仓库B的别名
    ```
