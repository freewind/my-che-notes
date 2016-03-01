Eclipse CHE 笔记
===============

重要资源
-------

1. 官网：<https://eclipse.org/che>
2. 代码：<https://github.com/eclipse/che>，另外在`eclipse`名下还有四五个以`che_`开头的仓库，也属于che
3. 文档：<https://eclipse-che.readme.io/docs/>，重中之重。CHE的架构非常灵活，里面有很多概念，在这些文档里对它们解释的非常清楚。在看这篇笔记的同时，一定要先把文档详细读一遍。
4. 我提的各种问题（有些还有价值，有些已经过时）：<https://github.com/eclipse/che/issues?utf8=%E2%9C%93&q=+author%3Afreewind+>

遇到的比较大的坑
-------------

### 1. tomcat启动慢

在digital ocean上创建的基于ubuntu 14.04的服务器上，不论使用docker还是java方式启动che，其中的tomcat经常（但不是每次）会特别慢，超过20分钟甚至一个小时才完成。

这除了浪费大量的时间外，还会导致很多奇怪的问题。比如在dashboard中创建project时，che会向创建的workspace machine中植入一个代理程序（一个tomcat+war）并启动它，然后等待少许后便访问其api来检查是否成功。由于tomcat启动的时间太长，会导致检查失败从而无法创建项目。

解决方法是在digital ocean上安装：

    apt-get install haveged

之后tomcat的启动时间将会缩短到10秒左右。

该问题的详细讨论在这里：https://www.digitalocean.com/community/questions/fresh-tomcat-takes-loong-time-to-start-up

该问题目前看起来是digital ocean上特有的问题，但也不能排除在其它的服务器上不会出现这个问题。所以如果遇到类似的问题，可以先看看这里。

### 2. 在远程服务器上启动che需要传入server ip

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


