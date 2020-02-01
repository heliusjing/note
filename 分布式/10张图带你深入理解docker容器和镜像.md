# 10张图带你深入理解Docker容器和镜像

这篇文章希望能够帮助读者深入理解Docker的命令，还有容器（container）和镜像（image）之间的区别，并深入探讨容器和运行中的容器之间的区别。

![1.png](10张图带你深入理解docker容器和镜像.assets/401405294bff9c66a1ec0e842391c856.png)


当我对Docker技术还是一知半解的时候，我发现理解Docker的命令非常困难。于是，我花了几周的时间来学习Docker的工作原理，更确切地说，是关于Docker统一文件系统（the union file system）的知识，然后回过头来再看Docker的命令，一切变得顺理成章，简单极了。

**题外话**：就我个人而言，掌握一门技术并合理使用它的最好办法就是深入理解这项技术背后的工作原理。通常情况下，一项新技术的诞生常常会伴随着媒体的大肆宣传和炒作，这使得用户很难看清技术的本质。更确切地说，新技术总是会发明一些新的术语或者隐喻词来帮助宣传，这在初期是非常有帮助的，但是这给技术的原理蒙上了一层砂纸，不利于用户在后期掌握技术的真谛。

Git就是一个很好的例子。我之前不能够很好的使用Git，于是我花了一段时间去学习Git的原理，直到这时，我才真正明白了Git的用法。我坚信只有真正理解Git内部原理的人才能够掌握这个工具。

### Image Definition

镜像（Image）就是一堆只读层（read-only layer）的统一视角，也许这个定义有些难以理解，下面的这张图能够帮助读者理解镜像的定义。

![2.png](10张图带你深入理解docker容器和镜像.assets/3b9079593f15903497a37c64a23b9c39.png)


从左边我们看到了多个只读层，它们重叠在一起。除了最下面一层，其它层都会有一个指针指向下一层。这些层是Docker内部的实现细节，并且能够在主机（译者注：运行Docker的机器）的文件系统上访问到。统一文件系统（union file system）技术能够将不同的层整合成一个文件系统，为这些层提供了一个统一的视角，这样就隐藏了多层的存在，在用户的角度看来，只存在一个文件系统。我们可以在图片的右边看到这个视角的形式。

你可以在你的主机文件系统上找到有关这些层的文件。需要注意的是，在一个运行中的容器内部，这些层是不可见的。在我的主机上，我发现它们存在于/var/lib/docker/aufs目录下。

```
sudo tree -L 1 /var/lib/docker//var/lib/docker/
├── aufs
├── containers
├── graph
├── init
├── linkgraph.db
├── repositories-aufs
├── tmp
├── trust
└── volumes
7 directories, 2 files
```



### Container Definition

容器（container）的定义和镜像（image）几乎一模一样，也是一堆层的统一视角，唯一区别在于容器的最上面那一层是可读可写的。

![3.png](10张图带你深入理解docker容器和镜像.assets/dacd660134e97cca60e04ba5fdcfa79e.png)


细心的读者可能会发现，容器的定义并没有提及容器是否在运行，没错，这是故意的。正是这个发现帮助我理解了很多困惑。

要点：容器 = 镜像 + 读写层。并且容器的定义并没有提及是否要运行容器。

接下来，我们将会讨论运行态容器。

### Running Container Definition

一个运行态容器（running container）被定义为一个可读写的统一文件系统加上隔离的进程空间和包含其中的进程。下面这张图片展示了一个运行中的容器。

![4.png](10张图带你深入理解docker容器和镜像.assets/1ccc2aa9e11e25e5ed2efd18cf1052c4.png)


正是文件系统隔离技术使得Docker成为了一个前途无量的技术。一个容器中的进程可能会对文件进行修改、删除、创建，这些改变都将作用于可读写层（read-write layer）。下面这张图展示了这个行为。

![5.png](10张图带你深入理解docker容器和镜像.assets/a20b70e3e4ca61faa2c3436e1bb2d93a.png)


我们可以通过运行以下命令来验证我们上面所说的：

```
docker run ubuntu touch happiness.txt
```


即便是这个ubuntu容器不再运行，我们依旧能够在主机的文件系统上找到这个新文件。

```
find / -name happiness.txt/var/lib/docker/aufs/diff/860a7b...889/happiness.txt
```



### Image Layer Definition

为了将零星的数据整合起来，我们提出了镜像层（image layer）这个概念。下面的这张图描述了一个镜像层，通过图片我们能够发现一个层并不仅仅包含文件系统的改变，它还能包含了其他重要信息。

![6.png](10张图带你深入理解docker容器和镜像.assets/39fb0f8e630b338bcca7a29da3acabb7.png)


元数据（metadata）就是关于这个层的额外信息，它不仅能够让Docker获取运行和构建时的信息，还包括父层的层次信息。需要注意，只读层和读写层都包含元数据。

![7.png](10张图带你深入理解docker容器和镜像.assets/f3218a8fcdd1fd8bcd5a719ca3c64f59.png)


除此之外，每一层都包括了一个指向父层的指针。如果一个层没有这个指针，说明它处于最底层。

![8.png](10张图带你深入理解docker容器和镜像.assets/53e377deeeb5f30bf939ed0836f851c9.png)



**Metadata Location：**
我发现在我自己的主机上，镜像层（image layer）的元数据被保存在名为”json”的文件中，比如说：

```
/var/lib/docker/graph/e809f156dc985.../json
```


e809f156dc985...就是这层的id

一个容器的元数据好像是被分成了很多文件，但或多或少能够在/var/lib/docker/containers/<id>目录下找到，<id>就是一个可读层的id。这个目录下的文件大多是运行时的数据，比如说网络，日志等等。

### 全局理解（Tying It All Together）

现在，让我们结合上面提到的实现细节来理解Docker的命令。



#### docker create <image-id>

![9.png](10张图带你深入理解docker容器和镜像.assets/062e7af0929dd205b2ac6efdd937d6f4.png)


docker create 命令为指定的镜像（image）添加了一个可读写层，构成了一个新的容器。注意，这个容器并没有运行。

![10.png](10张图带你深入理解docker容器和镜像.assets/162f02c6f9dce4ccdaa0c6c676da2196.png)



#### docker start <container-id>

![11.png](10张图带你深入理解docker容器和镜像.assets/1c38d4735e9760bdca025ff50a1b5386.png)


Docker start命令为容器文件系统创建了一个进程隔离空间。注意，每一个容器只能够有一个进程隔离空间。



#### docker run <image-id>

![12.png](10张图带你深入理解docker容器和镜像.assets/01aedf55bd21abbe607b3864d76f0ec0.png)


看到这个命令，读者通常会有一个疑问：docker start 和 docker run命令有什么区别。

![13.png](10张图带你深入理解docker容器和镜像.assets/275cc486d4ce7ecbdecf0ecc1de0a34b.png)


从图片可以看出，docker run 命令先是利用镜像创建了一个容器，然后运行这个容器。这个命令非常的方便，并且隐藏了两个命令的细节，但从另一方面来看，这容易让用户产生误解。

题外话：继续我们之前有关于Git的话题，我认为docker run命令类似于git pull命令。git pull命令就是git fetch 和 git merge两个命令的组合，同样的，docker run就是docker create和docker start两个命令的组合。

#### docker ps

![14.png](10张图带你深入理解docker容器和镜像.assets/f05a2a8dfc6641d8237306ff575aa283.png)


docker ps 命令会列出所有运行中的容器。这隐藏了非运行态容器的存在，如果想要找出这些容器，我们需要使用下面这个命令。

#### docker ps –a

![15.png](10张图带你深入理解docker容器和镜像.assets/d7f7b0ada7fc1c641c90745a959f9c05.png)


docker ps –a命令会列出所有的容器，不管是运行的，还是停止的。

#### docker images

![16.png](10张图带你深入理解docker容器和镜像.assets/d5b0b3e2e7acdcf35e7577d7670a46f7.png)


docker images命令会列出了所有顶层（top-level）镜像。实际上，在这里我们没有办法区分一个镜像和一个只读层，所以我们提出了top-level镜像。只有创建容器时使用的镜像或者是直接pull下来的镜像能被称为顶层（top-level）镜像，并且每一个顶层镜像下面都隐藏了多个镜像层。

#### docker images –a

![17.png](10张图带你深入理解docker容器和镜像.assets/ce083575e95a0e46b105c3596c12ca71.png)


docker images –a命令列出了所有的镜像，也可以说是列出了所有的可读层。如果你想要查看某一个image-id下的所有层，可以使用docker history来查看。

#### docker stop <container-id>

![18.png](10张图带你深入理解docker容器和镜像.assets/a41de8fe542efe25e3620691ad9238df.png)


docker stop命令会向运行中的容器发送一个SIGTERM的信号，然后停止所有的进程。

#### docker kill <container-id>

![19.png](10张图带你深入理解docker容器和镜像.assets/ac1cbc31d4f191e26e05fb1a11f04d26.png)


docker kill 命令向所有运行在容器中的进程发送了一个不友好的SIGKILL信号。



#### docker pause <container-id>

![20.png](10张图带你深入理解docker容器和镜像.assets/b701a3c51e7d1915da3bea0bc43efcd1.png)


docker stop和docker kill命令会发送UNIX的信号给运行中的进程，docker pause命令则不一样，它利用了cgroups的特性将运行中的进程空间暂停。具体的内部原理你可以在这里找到：[https://www.kernel.org/doc/Doc ... m.txt](https://www.kernel.org/doc/Documentation/cgroups/freezer-subsystem.txt)，但是这种方式的不足之处在于发送一个SIGTSTP信号对于进程来说不够简单易懂，以至于不能够让所有进程暂停。

#### docker rm <container-id>

![21.png](10张图带你深入理解docker容器和镜像.assets/8fa2d6e19c29f18548624efd64eb6dfa.png)


docker rm命令会移除构成容器的可读写层。注意，这个命令只能对非运行态容器执行。



#### docker rmi <image-id>

![22.png](10张图带你深入理解docker容器和镜像.assets/32a7f413a5f8ff936dd0f4a31d25fdcc.png)


docker rmi 命令会移除构成镜像的一个只读层。你只能够使用docker rmi来移除最顶层（top level layer）（也可以说是镜像），你也可以使用-f参数来强制删除中间的只读层。



#### docker commit <container-id>

![23.png](10张图带你深入理解docker容器和镜像.assets/3c02ccf4e7a2a353af065d93b26ae89e.png)


docker commit命令将容器的可读写层转换为一个只读层，这样就把一个容器转换成了不可变的镜像。

![24.png](10张图带你深入理解docker容器和镜像.assets/28059b3a499faba896263c0ff077fe3a.png)



#### docker build

![25.png](10张图带你深入理解docker容器和镜像.assets/b22cd304f28c715ae3ae6812476b222d.png)


docker build命令非常有趣，它会反复的执行多个命令。

![26.png](10张图带你深入理解docker容器和镜像.assets/100712263ecf4544dd11602adc39ee3e.png)


我们从上图可以看到，build命令根据Dockerfile文件中的FROM指令获取到镜像，然后重复地1）run（create和start）、2）修改、3）commit。在循环中的每一步都会生成一个新的层，因此许多新的层会被创建。



#### docker exec <running-container-id>

![27.png](10张图带你深入理解docker容器和镜像.assets/db64b3b38aff136d42e9ffeb81675bda.png)


docker exec 命令会在运行中的容器执行一个新进程。



#### docker inspect <container-id> or <image-id>

![28.png](10张图带你深入理解docker容器和镜像.assets/184f9d55770ca036cb5b1e6d96ce4a12.png)


docker inspect命令会提取出容器或者镜像最顶层的元数据。



#### docker save <image-id>

![29.png](10张图带你深入理解docker容器和镜像.assets/70cdbaf975c88bc83423d88be85476b5.png)


docker save命令会创建一个镜像的压缩文件，这个文件能够在另外一个主机的Docker上使用。和export命令不同，这个命令为每一个层都保存了它们的元数据。这个命令只能对镜像生效。



#### docker export <container-id>

![30.png](10张图带你深入理解docker容器和镜像.assets/1714c3dd524c807bf9c9b4d0fbe4d056.png)


docker export命令创建一个tar文件，并且移除了元数据和不必要的层，将多个层整合成了一个层，只保存了当前统一视角看到的内容（译者注：expoxt后的容器再import到Docker中，通过docker images –tree命令只能看到一个镜像；而save后的镜像则不同，它能够看到这个镜像的历史镜像）。



#### docker history <image-id>

![31.png](10张图带你深入理解docker容器和镜像.assets/b513dd5467f23fdd23523f60242d5dcb.png)


docker history命令递归地输出指定镜像的历史镜像。



### 结论

我希望你们能喜欢这篇文章。还有其他许多的命令（pull，search，restart，attach等）我没有提及，但是我相信通过阅读这篇文章，大部分的Docker命令都能够被很好理解。我仅仅学习了Docker两个星期，因此，如果我有什么地方说的不好，欢迎大家指出。