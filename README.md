Eclipse CHE 笔记
===============

写在最前面：填坑大法
-----------------

由于che目前处于紧张的开发之中，每天的改变（以及不兼容改变）都很多，很容易出现问题。如果你发现在使用che过程中遇到了不正常的情况（比如创建项目失败，进不了IDE等），请按这里的步骤一一尝试：

1. 如果che启动时间过长，检查是否安装了`haveged`(`apt-get install haveged`)
2. 检查启动che的命令是否有变化，比如某个参数名变了，参看<https://eclipse-che.readme.io/docs/usage-docker>
2. 在服务器上删除`/home/user/che`（里面保存的是之前创建的workspace及项目文件）后再启动che，否则有可能因为文件格式发生变化而出错
3. 在服务器上删除相应的docker images，比如`codenvy/che:nightly`，重新下载
4. 浏览器使用隐身模式，以防止js等文件因为缓存而出错

注意上面列出来的每一条都是真实发生过的，并且每一个都浪费了很多时间。

最后提醒的是，每当你发现了一个可以完美运行的`codenvy/che:nightly`，最好马上把它备份一下（比如到hub.docker.com中），这样如果che哪天出了问题，你至少还能找到一个可用的版本。

重要资源
-------

1. 官网：<https://eclipse.org/che>
2. 代码：<https://github.com/eclipse/che>，另外在`eclipse`名下还有四五个以`che_`开头的仓库，也属于che
3. 文档：<https://eclipse-che.readme.io/docs/>，重中之重。CHE的架构非常灵活，里面有很多概念，在这些文档里对它们解释的非常清楚。在看这篇笔记的同时，一定要先把文档详细读一遍。
4. 我提的各种问题（有些还有价值，有些已经过时）：<https://github.com/eclipse/che/issues?utf8=%E2%9C%93&q=+author%3Afreewind+>

其它更多的内容参看仓库中的其它文档
----------------------------

















