---
layout: post
header-img: img/laravel.jpg
title: laravel安装
tags: laravel
---

# 概述
最近有项目要用PHP开发，以前用过 [yii framework](http://www.yiiframework.com/)， 不过现在发现大家都推荐 [Laravel](https://laravel.com/)，大概看了下这个框架， 发现确实不错，所以就决定用它开发新项目。这篇文章记录了Laravel的安装过程。

# 安装
基于PHP的项目一般推荐用 [LAMP](https://zh.wikipedia.org/wiki/LAMP) 环境，可以在Linux下配置Apache（Nginx），Mysql，PHP还是比较麻烦的，所有Laravel提供了一个虚拟机镜像来帮助我们省去安装和配置环境的烦恼。Laravel提供了一个基于 [Vagrant][vagrant] 的镜像，支持 [VirtualBox](https://www.virtualbox.org/wiki/Downloads) 和 [VMWare](http://www.vmware.com/) 两种虚拟机软件。 Laravel提供的这个镜像叫 [Laravel Homestead](https://laravel.com/docs/5.2/homestead)。当然Laravel也支持手工安装，不过那样比较麻烦，现在以 Homestead 方式安装Laravel。
本文涉及的一些软件，可以从 [百度云盘](http://pan.baidu.com/s/1dE0vBKt) 中下载，这里放的是现在（2016-03-01）最新的软件。

## Vagrant
安装 Laravel Homestead 的第一步是安装 [Vargant][vagrant]，根据不同的操作系统安装不同版本。安装很简单，下载安装文件直接安装即可。

## Homestead Vagrant box
安装好Vagrant后就可以通过它安装 Homestead Vagrant box 了。安装命令为：

    {% highlight bash %}
    vagrant box add laravel/homestead
    {% endhighlight %}

不过这种方式在国内实在是太慢了，一个镜像有1G左右。办法是先把镜像文件通过其它方式下载到本地，然后再安装，下载前先访问[这里](https://atlas.hashicorp.com/laravel/boxes/homestead) 找到最新版本，比如现在的最新版本是0.4.1，所以下载virtualbox的文件地址就是： https://atlas.hashicorp.com/laravel/boxes/homestead/versions/0.4.1/providers/virtualbox.box 。当然如果是0.4.1版本的话，上面分享的百度云盘中已包含，从云盘中下载最快。
通过本地方案时，还需要提供一个配置文件，用来说明安装类型，版本等。云盘中的metadata.json就是这个的文件，内容如下：

    {% highlight javascript %}
    {
      "name": "laravel/homestead",
      "versions": [{
          "version": "0.4.1",
          "providers": [{
            "name": "virtualbox",
            "url": "file://virtualbox.box"
          }]
      }]
    }
    {% endhighlight %}

安装时把 virtual.box 和 metadata.json 两个文件放在一起，然后通过以下命令安装：

    vagrant box add metadata.json

是否正确安装可以通过如下命令验证：

    $ vagrant box list
    laravel/homestead     (virtualbox 0.4.1)

如果正确列出了 homestead及版本，说明安装成功了。

## Homestead
下一步安装Homestead本身，安装很简单，先定位到要安装的目录，然后git克隆homestead即可。

    git clone https://github.com/laravel/homestead.git

然后进入homestead目录，执行初始化脚本：

    $ bash init.sh

如果是window，直接运行init.bat即可。初始化的动作会在用户目录下新建一个'.homestead'有目录，只面有一个 Homestead.yaml 的配置文件，用来配置Homestead。主要配置项包含：

### provider
provider指虚拟机的类型，可以是virtualbox，vmware_fusion, vmware_workstation. 我使用的是virtualbox。

    provider: virtualbox

### 共享文件夹
虚拟机提供和宿主共享文件夹的功能，这样就很方便在宿主中写程序，在虚拟机中跑程序。默认配置如下：

    folders:
        - map: ~/Code
          to: /home/vagrant/Code

默认是把宿主的用户根目录下的 Code 文件夹，映射到虚拟机的 /home/vagrant/Code 目录中。这两个目录可以根据需要修改， 但要保证文件夹已经存在，要不然后面配置启动虚拟机时会报错。

### nginx站点配置
Homestead允许通过配置的方式，自动创建nginx站点，格式如下：

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public

也可以加上 [HHVM](http://hhvm.com/) 的支持:

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          hhvm: true

上面的例子，创建了一个域名（homestead.app)到目录 /home/vagrant/Code/Laravel/public 的站点配置。
如果后面sites节点有改动，需要执行以下命令更新虚拟机上的nginx配置。

    vagrant reload --provision

为了通过域名访问站点，你还需要修改宿主机器的hosts，添加如下记录：

    192.168.10.10   homestead.app

windows的hosts文件目录为： C:\Windows\System32\drivers\etc\hosts,  linux的hosts文件目录为： /etc/hosts

192.168.10.10 是虚拟机使用的IP，不影响宿主环境，建议不要修改。 这样就可以通过  http://homestead.app 访问站点了。 

## 启动Vagrant box
在homestead目录中，运行以下命令来启动虚拟机：

    vagrant up

如果顺利的话，虚拟机已经在运行了，可以打开virtualbox查看到正在运行。默认是没有显示运行界面，可以点击‘显示’按钮显示出来。虚拟机的登录用户和密码都是**vagrant**。 当然也可以通过ssh的方式登录， 直接运行以下命令：

    vagrant ssh

到这里虚拟机相关的安装和配置已完成。接下来是安装Laravel本身了。

# 安装 Laravel
Laravel通过 [composer](http://getcomposer.org/) 来安装，composer在虚拟机中已安装，可通过有两种方式安装。

## 通过 Laravel Installer
执行如下命令：

    composer global require "laravel/installer"

这样会安装一个 laravel 的命令。 不过这里有个bug，上面命令会创建一个 '~/.config/composer' 的目录，但 .profile 中写的path路径是 '~/.composer'， 导致 laravel 命令无法运行， 修改很简单，只要修改 .profile 文件中最后一行即可。

    PATH="/home/vagrant/.composer/vendor/bin:$PATH"
    改为
    PATH="/home/vagrant/.config/composer/vendor/bin:$PATH"

接着就可以用如下命令创建新项目了：

    laravel new blog

## 通过 composer  create-project
命令如下：

    composer create-project --prefer-dist laravel/laravel blog

这样就创建了一个名为blog的laravel应用程序了。

一切就绪，接下来就是开发了。


[vagrant]: https://www.vagrantup.com/ "Vargant"