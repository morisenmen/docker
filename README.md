docker
======

记录一些docker的相关知识

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。几乎没有性能开销,可以很容易地在机器和数据中心中运行。最重要的是,他们不依赖于任何语言、框架或包装系统。

docker的官网：http://www.docker.io， git地址：https://github.com/dotcloud/docker 。具体信息可以参见官网相关信息，我这里主要记录一些个人感受，仅供后来者参考。

docker的安装很简单。这里以ubuntu上面的安装为例。
    
    sudo apt-get update
    sudo apt-get install linux-image-extra-`uname -r`
    sudo sh -c "wget -qO- https://get.docker.io/gpg | apt-key add -" 
    sudo sh -c "echo deb http://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
    sudo apt-get update
    sudo apt-get install lxc-docker

以上安装docker的过程有任何不明白的地方，请自行百度或者google寻求解答。以上安装之后docker就已经可以使用了，但是每
次运行都得跟上`sudo`，很不方便，于是接下来进行如下操作。

    sudo groupadd docker
    sudo gpasswd -a {user} docker    #{user}为当前ubuntu登录用户
    sudo service docker restart
    docker version
    #若还未生效，则系统重启，则生效
    sudo reboot

现在的docker环境已经搭建好了，但是还不能使用，因为没有docker启动所需的image文件。接下来，我们需要去下载docker运行所需的image文件，官方有提供ubuntu的基本镜像，可以下载下来供以后的其他镜像作为基础镜像使用。

    docker pull ubuntu

下载下来之后就可以使用`docker run`命令启动ubuntu镜像了，但是当你启动之后你会发现这个镜像下面很多基本的命令都没有安装。没关系，接下来我们可以在这个最基本的镜像上安装我们所需要的应用。

接下来我们做一个简单的应用，我们使用这个ubuntu镜像来做一个redis服务器，并且映射到主机的`6379`端口上面。首先我们需要写一个redis.dockerfile。

    FROM        ubuntu
    RUN         apt-get update
    RUN         apt-get -y install redis-server
    EXPOSE      6379
    ENTRYPOINT  ["/usr/bin/redis-server"]

好了，我们的dockerfile文件编写好了，但是我们还没有docker所需要的image文件，接下来我们就用这个redis.dockerfile文件来生成我们redis服务器的image

    docker build -t {repository}/redis_server/{version} -< redis.dockerfile 
    #{repository}为docker库的地址，
    #redis_server为新建立的这个image的名字，
    #{version}为redis的版本

这样我们就建立好我们的redis服务器的镜像了，可以使用`docker images`查看已经有的images。接下来启动redis服务并映射到主机的6379端口上来。

    docker run -name redis_server -d -p 6379:6379 redis_server

好了，我们的redis服务器已经启动了。可以连接上来试试了。

    telnet localhost 6379
    Trying ::1...
    Connected to localhost.
    Escape character is '^]'.
    set docker awesome
    +OK
    get docker
    $7
    awesome
    get redis
    $-1
