---
title: Git SSH 配置
categories: git
tags: git
---

##### Tips: [Github about ssh](https://help.github.com/articles/about-ssh/)

#### 这两天在搭建Github博客，在配置ssh时遇到了一些坑，网上的很少写得齐全，第一篇博客就把这个坑填上吧。 

##### Tips: 本次配置在Windows环境下，使用git bash。Linux下命令基本一致。

## 步骤

### 1. 查看本地是否存在ssh
``` bash
$ cd ~/.ssh
$ ls -al ~/.ssh
```

### 2. 生成 ssh key

``` bash
$ ssh-keygen -t rsa 4096 -C "your_email@example.com"
```

``` bash
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/Hunter/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/Hunter/.ssh/id_rsa.
Your public key has been saved in /c/Users/Hunter/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:7KwlOZ4yljBZE2ZJ7dr8QGIyQeiPk49L+01fnC0hAZY your_email@example.com
The key's randomart image is:
+---[RSA 4096]----+
| o...=.          |
|. . *Eo          |
|.  + o .         |
| .o = o..        |
|  +* B .S.       |
| ++.. ++o +      |
| .+o o+o+= .     |
|....B..*o .      |
| ooo ++.         |
+----[SHA256]-----+
```

##### Tips: 双引号内填保存在github段的邮箱


### 3. 将key添加到ssh-agent(很重要)

#### 先确认ssh-agent是可用的

``` bash
$ eval $(ssh-agent -s)
```

``` bash
Agent pid 13140
```

#### 将ssh key添加到ssh-agent

``` bash
$ ssh-add ~/.ssh/id_rsa
```

```bash
Identity added: /c/Users/Hunter/.ssh/id_rsa (/c/Users/Hunter/.ssh/id_rsa)
```

### 4. 将ssh public key添加到github

#### 复制public key内容
```bash
$ clip < ~/.ssh/id_rsa.pub
```

#### 登录github->选择Settings->SSH keys->New SSH key
#### 将public key内容复制上去，并命名

### 5. 测试ssh key

``` bash
$ ssh -t git@github.com
```

``` bash
Warning: Permanently added the RSA host key for IP address '192.30.252.128' to the list of known hosts.
PTY allocation request failed on channel 0
```

#### 配置完毕
这时就可以进行clone等操作了
