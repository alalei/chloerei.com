---
layout: post
title: 写给大家看的 Rails 部署：第一篇 简单快捷的部署方案
---

## 前言

Rails 是当今最优秀的 Web 框架，它大大提升了 Web 开发者的开发效率。但由于 Web 实质上是一门综合技术，牵涉的领域较多，没有其他 Web 开发经验的人也是不好上手 Rails 的。

最近，新手问题中部署相关的问题出现频率较高。其实现在并不是没有部署相关的书籍和文章，比如入门宝书《[Web开发敏捷之道][1]》中就有一章的内容是专门讲部署。不过我观察发现，中文的资料普遍滞后，英文资料对于新手可能有难度。所以我萌生了写一系列、由浅至深的部署文章的想法。我的部署经验并不丰富，基本是伴随 CodeCampo 的上线运行积累起来的，很多时候处于得过且过的状态。如果能达到理解我的部署方案，并且找到更好的替代方案，那么就相当于越过了入门这道坎了，相信你就可以根据自己的需求优化自己的部署方案。

## 第一篇：简单快捷的基础部署

本篇希望讲解一个简单快捷的基础部署方案。不过简单快捷并不等于无需思考，每个操作的前后会有少量的解释。

Rails 的开发环境服务器非常容易跑起来，只要输入`rails s`，就能在本机的 3000 端口打开一个 web 服务。那么，真实的部署环境是不是`rails s` 就够了呢？当然不是，`rails s` 调用的是 Ruby 标准库里的 WEBrick 服务器，它并不足够支撑真实世界的大量请求。所以，我们需要一个 Rails 的**生产环境**部署方案。

如果现在打开 Google 搜索「Rails 部署」，你会看到五花八门的部署方案，一大堆名词映入你的眼帘：Apache、Nginx、Unicorn、Thin、Puma……这真让人看花了眼。之所以有这么多部署方案，是因为真实世界有各种各样的项目需求，也有各种各样的开发者喜好，这些五花八门的部署方案，也是 Rails 社区活跃的证明。

现在我希望你把焦点放在我推荐的部署方案上：Nginx + Passenger，毕竟，DHH 也推荐 Passenger，相信他没错！

> Phusion Passenger Enterprise is looking like a pretty swanky way to deploy Rails apps —— [DHH][2] (Creator of Ruby on Rails)

在本篇文章里，你会接触到以下工具：

 1. RVM
 2. Nginx
 3. Passenger

没用过不要紧，下面会一个个解释它们的用途。本文会使用纯净的 ubuntu server 12.04 做范例，从零开始搭建 Rails 生产环境。

### RVM

[RVM][3](Ruby Version Manager) 是一个多版本 Ruby 安装管理工具。为什么会需要安装多版本的 Ruby 呢？最基本的，这是为了升级 Ruby 版本的需求。即使在一般的 Linux 发行版，可以通过软件源升级 Ruby，但是其版本通常会落后官方版本不少。为了更好的效率和更少的 bug，推荐使用尽量新的 Ruby 版本。

实际上，希望你在一开始安装 Ruby 学习的时候就开始使用 RVM，而不仅仅是部署的时候。

注：如果你使用 Ubuntu 桌面，先要进行这个设置 https://rvm.io/integration/gnome-terminal/

#### 安装 RVM

RVM 的安装有官方脚本的支持，只需要输入以下命令

    curl -L https://get.rvm.io | sudo bash -s stable
    source /etc/profile.d/rvm.sh
    sudo gpasswd -a rei rvm
    

这里使用了`sudo`命令去安装 Multi-User 模式的 RVM，换句话说就是系统级别的 RVM，所有 Linux 用户都可使用。之所以使用 Multi-User 模式，是因为很多时候我还会创建新用户用于隔离权限，整个系统使用同一份 RVM 无疑会带来很多便利。

接着，需要安装一些系统依赖。输入以下命令

    rvm requirements

rvm 会将不同 ruby 实现所需的系统包列出，这里使用 Matz 的官方 Ruby（MRI），要安装以下的包

    sudo apt-get install build-essential openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison subversion

然后，通过 RVM 安装 Ruby

    rvmsudo rvm install ruby

最后，设定 ruby 默认版本

    rvm use --default 1.9.3

检查当前 Ruby 版本

    ruby -v

第一步安装 Ruby 已经完成，借下来开始安装本篇部署的另一个主角：Passenger。

### Nginx + Passenger

你应该多少听过 Nginx 的名字，这是一个高性能的 HTTP 前端服务器。Nginx 一般用来做静态服务器或者反向代理，不具有执行应用的功能。而 Passenger 是一个可用于生产环境的 Rails 应用服务器，其提供了 Apache 和 Nginx 的版本，将它编译进 Nginx 就可以给 Nginx 添加运行 Rails 应用的功能。

Passenger 提供了非常人性化的安装脚本，即使没有 Nginx 编译经验的人也能驾轻就熟（比如我）。

首先，通过 rubygems 安装 passenger

    gem install passenger

然后，执行 passenger nginx 模块的安装脚本

    rvmsudo passenger-install-nginx-module

安装脚本会开始引导编译 nginx 版本 passenger，根据提示进行操作既可。

如果你是一路跟着教程操作的，会发现 passenger 检查出一个问题

    Checking for required software...
    
     * GNU C++ compiler... found at /usr/bin/g++
     * The 'make' tool... found at /usr/bin/make
     * A download tool like 'wget' or 'curl'... found at /usr/bin/wget
     * Ruby development headers... found
     * OpenSSL support for Ruby... found
     * RubyGems... found
     * Rake... found at /usr/local/rvm/wrappers/ruby-1.9.3-p194/rake
     * rack... found
     * Curl development headers with SSL support... not found
     * OpenSSL development headers... found
     * Zlib development headers... found
    
    Some required software is not installed.
    But don't worry, this installer will tell you how to install them.

问题是系统的 Curl 没有提供 SSL 支持，按回车继续，passenger 安装脚本给出了解决方案，安装 ubuntu 的一个包既可

    sudo apt-get install libcurl4-openssl-dev

安装完成后再次执行`rvmsudo passenger-install-nginx-module`，会提示是完整安装(1)还是自定义安装(2)，因为我没有另外的模块编译的需要，所以输入最简单的 1 然后回车。然后安装脚本就会自动下载所需的源码包进行编译。编译过程会询问安装路径，使用默认的既可。

很快，nginx 就会编译完成，安装脚本在退出前还会列出 nginx 配置的范例，这也是随后要做的。

现在再添加一个 nginx 的启动脚本，方便我们用 ubuntu server 的方式管理 nginx 服务进程。执行以下命令

    cd /etc/init.d
    sudo wget https://raw.github.com/chloerei/nginx-init-ubuntu-passenger/master/nginx
    sudo update-rc.d nginx defaults
    sudo chmod +x nginx
    sudo service nginx start

以后可以用`sudo service nginx [start|stop|restart]`命令方便的重启 nginx 服务。

### nginx 配置

现在打开 nginx 的配置文件`sudo vi /opt/nginx/conf/nginx.conf`，里面已经有了一些初始配置，忽略注释和无关的配置，关键部分如下

    ...
    http {
        passenger_root /usr/local/rvm/gems/ruby-1.9.3-p194/gems/passenger-3.0.15;
        passenger_ruby /usr/local/rvm/wrappers/ruby-1.9.3-p194/ruby;
    
        ...
    
        server {
            listen       80;
            server_name  localhost;
            ...
        }
    }

关注 passenger_root 和 passenger_ruby 这两行，这是让 nginx 找到 passenger 执行路径的配置。配置文件还默认添加了一个 server 配置块，添加一个叫 localhost 的主机，这个配置块对我们无用可以删除。

接着来配置我们的 Rails 应用。假设你的应用源码放在 `/home/webuser/codecampo.com` 下面，并且已经有一个 codecampo.com 的域名指向你的服务器（如果你只是在本地演练，简单起见可以先用 localhost 域名），那么添加一个 server 配置块，如下所示

    ...
    http {
        passenger_root /usr/local/rvm/gems/ruby-1.9.3-p194/gems/passenger-3.0.15;
        passenger_ruby /usr/local/rvm/wrappers/ruby-1.9.3-p194/ruby;
    
        ...
    
        server {
            listen       80;
            server_name  codecampo.com; # 如果是本地演练可以设置为 localhsot
            root /home/webuser/codecampo.com/public;
            passenger_enabled on;
        }
    }

重启 nginx 服务

    sudo service nginx restart

打开浏览器，输入网址 codecampo.com （或者 localhost），看是否能成功启动你的 Rails 应用？

### 哎呀报错了！

很大几率，你在部署第一个 Rails 应用的时候还是遇到问题，这样你应该会看到一个很炫的 Passenger 错误页面（好吧，至少比起丑陋的错误页面让你心情舒服不少）。

![alt text][4]

出现这个页面的时候，至少说明 Passenger 部署环境已经安装好了，接下来的工作和你的开发环境差不多。常见的问题有：

 * 项目所需的 ruby gems 没有安装（bundle install）
 * 数据库没有安装（sudo apt-get install mysql-server/mongodb）
 * javascript runtime 没有安装（`sudo apt-get install nodejs）
 * 没有运行数据迁移（rake db:migrate）
 * 没有编译 js/css （rake assets:precompile）

从上图的 Error Message 来看，应用程序找不到 bcrypt-ruby 这个包，也就是需要运行 bundle 了。根据你的项目所依赖的工具不同，遇到的问题也不同，只有具体问题具体分析了。

你可以在项目目录下运行

    rails s -e production

来测试生产环境是否准备就绪，然后让 passenger 重启应用

    touch tmp/restart.txt

## 总结

本篇教程展示了由一台 Ubuntu server 从零开始搭建 Rails 应用部署环境的过程。相比本地开发环境，服务端的生产环境的配置固然复杂，但 web 应用一次部署、随处可用，以及可以让开发者随心所欲选择工具而不用顾虑客户端环境的好处会在日后显现出来。当你第一次成功搭建起部署环境的时候，你就打开了一个无限可能的新世界的大门。

下篇预告：自动化部署。

  [1]: http://book.douban.com/subject/10528446/
  [2]: https://twitter.com/dhh/status/230706337288425472
  [3]: https://rvm.io/
  [4]: http://i.minus.com/ibiqvfdoqoACnt.png
