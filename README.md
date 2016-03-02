Eclipse CHE 笔记
===============

重要资源
-------

1. 官网：<https://eclipse.org/che>
2. 代码：<https://github.com/eclipse/che>，另外在`eclipse`名下还有四五个以`che_`开头的仓库，也属于che
3. 文档：<https://eclipse-che.readme.io/docs/>，重中之重。CHE的架构非常灵活，里面有很多概念，在这些文档里对它们解释的非常清楚。在看这篇笔记的同时，一定要先把文档详细读一遍。
4. 我提的各种问题（有些还有价值，有些已经过时）：<https://github.com/eclipse/che/issues?utf8=%E2%9C%93&q=+author%3Afreewind+>

服务器安装前的准备工作
--------------------

### 1. 操作系统请选择ubuntu 14.04 x64

因为我们在该版本的操作系统上进行了大量的尝试，解决了很多问题。如果使用其它系统，可能会遇到不同的问题。

### 2. 添加一个非root用户

由于安全原因，及后面安装`docker`的需要，我们需要添加一个非root用户，这里使用`twer`这个名字。

```
adduser twer
adduser twer sudo
passwd twer
```

然后根据提示输入密码。

由于后面的操作将在`twer`下进行，所以我们要转到该帐户下。

```
su twer
```

### 3. 安装docker


```
wget -qO- https://get.docker.com/ | sh
```


需要将`twer`这个用户加入到`docker`组中，方便操作：

```
sudo usermod -aG docker twer
```

注意，你必须退出`twer`再重新登录进来才生效。

然后检查是否安装成功：

```
docker run hello-world
```

如果一切顺利的话，你会看到一些执行日志及运行成功的提示信息。

### 4. 设置文件的最大可连接数

很多系统默认的“文件最大可连接数”以及“最大进程数”等都比较小(`1024`)，导致che在运行过程中，很容易出现`Too many open files`的错误，无法正常运行。

使用以下命令查看：

```
ulimit -a
```

我们可以给它设置一个比较大的值：

```
sudo vi /etc/security/limits.conf
```

在最后加入以下行：

```
*                hard    nofile          100000
*                soft    nofile          100000

*                hard    nproc           100000
*                soft    nproc           100000
```

这样其数值设为了`100000`，基本够用。

### 5. 设置交换内存

che对于服务器的CPU和内存要求都比较高。这是因为每当建立一个workspace，che都需要以docker container的方式为它创建一个workspace machine，并且每个machine里都要运行一个tomcat8启动的代理程序。并且由于workspace machine由于是运行项目代码用的，其内存将根据项目类型和规模有不同的要求。比如对于js项目，内存设为`128M`即可，但对于Java项目，可能得`1G`以上。

通常服务商提供的服务器，其交换内存默认为0，导致内存基本不够用。

使用`top`命令查看：

```
top - 07:12:18 up 22:25,  3 users,  load average: 1.79, 0.62, 0.26
Tasks: 112 total,   3 running, 109 sleeping,   0 stopped,   0 zombie
%Cpu(s): 91.7 us,  8.0 sy,  0.0 ni,  0.0 id,  0.3 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:   2048484 total,  1888816 used,   159668 free,     1232 buffers
KiB Swap:        0 total,        0 used,        0 free.   304788 cached Mem
```

`KiB Mem`那行显示我们有2G的内存，最后一行`KiB Swap`显示交换内存为`0`。

我们可以按以下方式给服务器增加`4G`左右的内存：

```
# 4G
sudo mkdir -p /media/fasthdd/
sudo dd if=/dev/zero of=/media/fasthdd/swapfile.img bs=400 count=10M
sudo mkswap /media/fasthdd/swapfile.img
# Add this line to /etc/fstab /media/fasthdd/swapfile.img swap swap sw 0 0
sudo swapon /media/fasthdd/swapfile.img
```

另外需要注意的是，对于che server来说，CPU和内存都是瓶颈。我们在digital的`4G RAM`的主机上测试，最多只能同时跑7个最简单的workspace machine，再多的话，就算内存够用，CPU也支持不了了，这时就需要换更强的CPU了。

### 6. 安装haveged

在某些服务器上（如digital ocean）启动che，其中的tomcat经常（但不是每次）会特别慢，超过20分钟甚至一个小时才完成。

这除了浪费大量的时间外，还会导致很多奇怪的问题。比如在dashboard中创建project时，che会向创建的workspace machine中植入一个代理程序（一个tomcat+war）并启动它，然后等待少许后便访问其api来检查是否成功。由于tomcat启动的时间太长，会导致检查失败从而无法创建项目。

解决方法是在digital ocean上安装：

    apt-get install haveged

之后tomcat的启动时间将会缩短到10秒左右。

该问题的详细讨论在这里：https://www.digitalocean.com/community/questions/fresh-tomcat-takes-loong-time-to-start-up

### 现在可以用docker跑che了

经过以上的配置，我们就可以在这台服务器上以docker的形式来运行che了。

但是如果我们想在该服务器上以本地Java程序的方式运行che，或者有时候还想编译一下che等，这时我们还需要安装以下软件：

### 7. 安装Java

必须是jdk1.8或以上

```
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt-get update
$ sudo apt-get install oracle-java8-installer
```

检查java版本：

```
java -version
```


### 8. 安装Maven

目前需要3.3.x

```
mkdir downloads
cd downloads
wget http://mirrors.sonic.net/apache/maven/maven-3/3.3.3/binaries/apache-maven-3.3.3-bin.tar.gz
tar -zxf apache-maven-3.3.3-bin.tar.gz
sudo cp -R apache-maven-3.3.3 /usr/local
sudo ln -s /usr/local/apache-maven-3.3.3/bin/mvn /usr/bin/mvn
```

检查mvn版本：

```
mvn --version
```

遇到的一些坑以及需要注意的问题
-------------------------

### 1. 在远程服务器上启动che需要传入server ip

这又是一个需要细读文档才能发现的问题。如果我们在远程linux服务器上启动che时，需要使用一个特定的参数传入server ip才能让che正常运行。其原因是当che使用docker创建了一些workspace machine后，它们需要该参数才能正确的找到che server。

- 如果使用docker的方式，则需要在命令行中加入`-e DOCKER_MACHINE_HOST=<server_ip>`，比如：

    ```
    docker run 
        ...
        -e DOCKER_MACHINE_HOST=198.199.105.97
        codenvy/che
    ```

- 如果使用`che.sh`脚本启动，则需要传入`-r:<server_ip>`，比如：

    ```
    che.sh -r:198.199.105.97 run
    ```

但在本地（比如mac/windows上），由于docker只能以虚拟机的方式运行，che事先把该参数设好了，所以不需要传入它们也能正常运行。这就是为什么在本地一切正常，但是在远程就不行的原因。

另外缺少该参数后，che只是在创建project的时候才会出现一些不是特别明显的错误。如果当你创建project时，看到它的输出中有一些类似

```
Not able to connect to ws://che-host:8080/che/api/eventbus/?token=dummy_token because Connection timeout. Retrying
```

这样的错误时，需要看看是不是这个原因。

mvn编译che时，提示`OutOfMemoryError`
----------------------------------

这是由于mvn默认使用的内存比较小，编译che时不够用，我们可以给它设大一些：

```
export MAVEN_OPTS="-Xmx1024m -XX:MaxPermSize=256m"
```
