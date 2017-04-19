---
layout: post
header-img: img/gitlab.png
title: 基于 gitlab 的持续集成2
tags: gitlab,ci
---

# 概述
目前使用 .net core 作为服务后台，使用 dotnet 的 runtime 作为 docker 来运行程序，运行命令类似：

```shell
dotnet xxx.dll
```

现在有个问题是要更新程序必须要先停止运行再重新开启，这样很不方便。

目前的解决方案是在 gitlab ci 的 runner docker (dotnet sdk) 中，先执行正常的编辑，复制等操作，完成后再通过 ssh 到 主机把 docker 重新启动一下。
gitlab-ci 的配置文件类似如下：

```
before_script:
  - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
  - mkdir -p ~/.ssh
  - eval $(ssh-agent -s)
  - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'

stages:
  - build

build:
  artifacts:
    paths:
    - build/
  stage: build
  script:
    - cd Regional.Web
    - dotnet restore
    - dotnet publish -c release
    - cp -R /builds/maxwell/lifecycle-regional/Regional.Web/bin /root/lifecycle-regional/Regional.Web/
    - ssh-add <(echo "$STAGING_PRIVATE_KEY")
    - ssh -p22 user@server_ip "docker restart web-docker-name"
```

在 before_script 中执行如下操作：

1. 安装 ssh 客户端，如果没有安装的话。
2. 创建 .ssh 目录。
3. 启动 ssh-agent
4. 配置  ssh/config 设置 StrictHostKeyChecking 为 no，这样 ssh 时就不会提示是否要连接的提示。

在 build stage 中执行正常的 dotnet restore 和 dotnet publish 操作。完成后把publish结果得到另一个目录中，该目录通过 docker 的数据卷共享。
接着把之前在 gitlab 中设置的服务器私有ssh key添加到当前的 ssh-agent 中。然后 ssh 到 服务器把 web 的 docker 重启。
其中

```shell
ssh-add <(echo "$STAGING_PRIVATE_KEY")
```

的操作是为了 ssh 到服务器时不需要输入密码。
如果遇到 “Enter passphrase for /dev/fd/63” 的问题，请参考 [这里](https://gitlab.com/gitlab-examples/ssh-private-key/issues/1)

使用 ssh 需要在服务器上做一些操作，正常的执行流程如下：

1. ssh-keygen: no password
2. cat ~/.ssh/id_rsa: 复制输入的私有 ssh key
3. cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys  把 ssh 公钥添加到认证授权文件，这样可以不需要输入密码
4. 复制 ssh 私有密钥到 GitLab 服务中定义的私有变量中。

参考： [https://docs.gitlab.com/ce/ci/examples/deployment/composer-npm-deploy.html](https://docs.gitlab.com/ce/ci/examples/deployment/composer-npm-deploy.html)
[https://gitlab.com/gitlab-examples/ssh-private-key/issues/1](https://gitlab.com/gitlab-examples/ssh-private-key/issues/1)

完。
