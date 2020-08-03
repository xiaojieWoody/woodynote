* **容器，其实是一种特殊的进程而已**
  * **Linux 操作系统提供了PID、 Mount、UTS、IPC、Network 和 User 这些 Namespace，用来对各种不同的进程上下文进行“障眼法”操作**
    * Mount Namespace，用于让被隔离进程只看到当前 Namespace 里的挂载点信息
    * Network Namespace，用于让被隔离进程看到当前 Namespace 里的网络设备和配置
  * 实际上是在创建容器进程时，指定了这个进程所需要启用的一组 Namespace 参数。这样，容器就只能“看”到当前 Namespace 所限定的资源、文件、设备、状态，或者配置。而对于宿主机以及其他不相关的程序，它就完全看不到了
  
* Linux 容器中用来实现“隔离”的技术手段：Namespace。**Namespace 技术实际上修改了应用进程看待整个计算机“视图”，即它的“视线”被操作系统做了限制，只能“看到”某些指定的内容**。但对于宿主机来说，这些被“隔离”了的进程跟其他进程并没有太大区别

* 虚拟机与容器技术的对比图里，不应该把 Docker Engine 或者任何容器管理工具放在跟 Hypervisor 相同的位置，因为它们并不像 Hypervisor 那样对应用进程的隔离环境负责，也不会创建任何实体的“容器”，真正对隔离环境负责的是宿主机操作系统本身
  
  * 在这个对比图里，应该把 Docker 画在跟应用同级别并且靠边的位置。这意味着，用户运行在容器里的应用进程，跟宿主机上的其他进程一样，都由宿主机操作系统统一管理，只不过这些被隔离的进程拥有额外设置过的 Namespace 参数。而 Docker 项目在这里扮演的角色，更多的是旁路式的辅助和管理工作
  
* **“敏捷”和“高性能”是容器相较于虚拟机最大的优势**
  * 使用虚拟化技术作为应用沙盒，就必须要由 Hypervisor 来负责创建虚拟机，这个虚拟机是真实存在的，并且它里面必须运行一个完整的 Guest OS 才能执行用户的应用进程。这就不可避免地带来了额外的资源消耗和占用
    * 根据实验，一个运行着 CentOS 的 KVM 虚拟机启动后，在不做优化的情况下，虚拟机自己就需要占用 100~200 MB 内存。此外，用户应用运行在虚拟机里面，它对宿主机操作系统的调用就不可避免地要经过虚拟化软件的拦截和处理，这本身又是一层性能损耗，尤其对计算资源、网络和磁盘 I/O 的损耗非常大
  * 而相比之下，容器化后的用户应用，却依然还是一个宿主机上的普通进程，这就意味着这些因为虚拟化而带来的性能损耗都是不存在的；而另一方面，使用 Namespace 作为隔离手段的容器并不需要单独的 Guest OS，这就使得容器额外的资源占用几乎可以忽略不计
  
* 问题：**隔离得不彻底**
  * 首先，既然容器只是运行在宿主机上的一种特殊的进程，那么多个容器之间使用的就还是同一个宿主机的操作系统内核
    * 尽管可以在容器里通过 Mount Namespace 单独挂载其他不同版本的操作系统文件，比如 CentOS 或者 Ubuntu，但这并不能改变共享宿主机内核的事实。这意味着，如果要在 Windows 宿主机上运行 Linux 容器，或者在低版本的 Linux 宿主机上运行高版本的 Linux 容器，都是行不通的
  * 其次，在 Linux 内核中，有很多资源和对象是不能被 Namespace 化的，最典型的例子就是：时间
    * 如果你的容器中的程序使用 settimeofday(2) 系统调用修改了时间，整个宿主机的时间都会被随之修改
  
* **容器的“限制”问题**

  * 表面上被隔离了起来，但是它所能够使用到的资源（比如 CPU、内存），却是可以随时被宿主机上的其他进程（或者其他容器）占用的。当然，它自己也可能把所有资源吃光

  * **Linux Cgroups 就是 Linux 内核中用来为进程设置资源限制的一个重要功能**

    * **最主要的作用，就是限制一个进程组能够使用的资源上限，包括 CPU、内存、磁盘、网络带宽等等**

    * Cgroups 还能够对进程进行优先级设置、审计，以及将进程挂起和恢复等操作

    * 在 /sys/fs/cgroup 下面有很多诸如 cpuset、cpu、 memory 这样的子目录，也叫子系统。这些都是这台机器当前可以被 Cgroups 进行限制的资源种类。而在子系统对应的资源种类下，就可以看到该类资源具体可以被限制的方法

      * 除 CPU 子系统外
      * blkio，为块设备设定I/O 限制，一般用于磁盘等设备
      * cpuset，为进程分配单独的 CPU 核和对应的内存节点
      * memory，为进程设定内存使用的限制

    * **就是一个子系统目录加上一组资源限制文件的组合**。而对于 Docker 等 Linux 容器项目来说，它们只需要在每个子系统下面，为每个容器创建一个控制组（即创建一个新目录），然后在启动容器进程之后，把这个进程的 PID 填写到对应控制组的 tasks 文件中就可以了，而至于在这些控制组下面的资源文件里填上什么值，就靠用户执行 docker run 时的参数指定了

      ```shell
      docker run -it --cpu-period=100000 --cpu-quota=20000 ubuntu /bin/bash
      # 在启动这个容器后，可以通过查看 Cgroups 文件系统下，CPU 子系统中，“docker”这个控制组里的资源限制文件的内容来确认
      $ cat /sys/fs/cgroup/cpu/docker/5d5c9f67d/cpu.cfs_period_us 
      100000
      # 意味着这个 Docker 容器，只能使用到 20% 的 CPU 带宽
      $ cat /sys/fs/cgroup/cpu/docker/5d5c9f67d/cpu.cfs_quota_us 
      20000s
      ```

    * Cgroups 对资源的限制能力也有很多不完善的地方，被提及最多的自然是 /proc 文件系统的问题

      * 众所周知，Linux 下的 /proc 目录存储的是记录当前内核运行状态的一系列特殊文件，用户可以通过访问这些文件，查看系统以及当前正在运行的进程的信息，比如 CPU 使用情况、内存占用率等，这些文件也是 top 指令查看系统信息的主要数据来源
      * 如果在容器里执行 top 指令，就会发现，它显示的信息居然是宿主机的 CPU 和内存数据，而不是当前容器的数据
      * 造成这个问题的原因就是，/proc 文件系统并不知道用户通过 Cgroups 给这个容器做了什么样的资源限制，即：/proc 文件系统不了解 Cgroups 限制的存在
      * 在生产环境中，这个问题必须进行修正，否则应用程序在容器里读取到的 CPU 核数、可用内存等信息都是宿主机上的数据，这会给应用的运行带来非常大的困惑和风险。这也是在企业中，容器化应用碰到的一个常见问题，也是容器相较于虚拟机另一个不尽如人意的地方
      * 如何修复容器中的 top 指令以及 /proc 文件系统中的信息呢？（提示：lxcfs）

* **容器是一个“单进程”模型**

  * 一个正在运行的 Docker 容器，其实就是一个启用了多个 Linux Namespace 的应用进程，而这个进程能够使用的资源量，则受 Cgroups 配置的限制
  * 由于一个容器的本质就是一个进程，用户的应用进程实际上就是容器里 PID=1 的进程，也是其他后续创建的所有进程的父进程。这就意味着，在一个容器中，你没办法同时运行两个不同的应用，除非你能事先找到一个公共的 PID=1 的程序来充当两个不同应用的父进程，这也是为什么很多人都会用 systemd 或者 supervisord 这样的软件来代替应用本身作为容器的启动进程
  * 容器本身的设计，就是希望容器和应用能够**同生命周期**

* **容器里的进程看到的文件系统又是什么样子的呢？**

  * 是一个关于 Mount Namespace 的问题：容器里的应用进程，理应看到一份完全独立的文件系统。这样，它就可以在自己的容器目录（比如 /tmp）下进行操作，而完全不会受宿主机以及其他容器的影响

  * 即使开启了 Mount Namespace，容器进程看到的文件系统也跟宿主机完全一样

  * **Mount Namespace 修改的，是容器进程对文件系统“挂载点”的认知**。但是，这也就意味着，只有在“挂载”这个操作发生之后，进程的视图才会被改变。而在此之前，新创建的容器会直接继承宿主机的各个挂载点

  * 一个解决办法：创建新进程时，除了声明要启用 Mount Namespace 之外，还可以告诉容器进程，有哪些目录需要重新挂载，就比如这个 /tmp 目录。于是，在容器进程执行前可以添加一步重新挂载 /tmp 目录的操作

  * 创建的新进程启用了 Mount Namespace，所以这次重新挂载的操作，只在容器进程的 Mount Namespace 中有效。如果在宿主机上用 mount -l 来检查一下这个挂载，会发现它是不存在的

  * **这就是 Mount Namespace 跟其他 Namespace 的使用略有不同的地方：它对容器进程视图的改变，一定是伴随着挂载操作（mount）才能生效**

  * 每当创建一个新容器时，我希望容器进程看到的文件系统就是一个独立的隔离环境，而不是继承自宿主机的文件系统。怎么才能做到这一点呢？

    * 可以在容器进程启动之前重新挂载它的整个根目录“/”。而由于 Mount Namespace 的存在，这个挂载对宿主机不可见，所以容器进程就可以在里面随便折腾了

    * 在 Linux 操作系统里，有一个名为 chroot 的命令可以帮助你在 shell 中方便地完成这个工作。顾名思义，它的作用就是帮你“change root file system”，即改变进程的根目录到你指定的位置。它的用法也非常简单

    * 假设，我们现在有一个 $HOME/test 目录，想要把它作为一个 /bin/bash 进程的根目录

      ```shell
      # 首先，创建一个 test 目录和几个 lib 文件夹：
      $ mkdir -p $HOME/test
      $ mkdir -p $HOME/test/{bin,lib64,lib}
      $ cd $T
      # 然后，把 bash 命令拷贝到 test 目录对应的 bin 路径下：
      $ cp -v /bin/{bash,ls} $HOME/test/bin
      # 接下来，把 bash 命令需要的所有 so 文件，也拷贝到 test 目录对应的 lib 路径下。找到 so 文件可以用 ldd 命令
      $ T=$HOME/test
      $ list="$(ldd /bin/ls | egrep -o '/lib.*\.[0-9]')"
      $ for i in $list; do cp -v "$i" "${T}${i}"; done
      # 最后，执行 chroot 命令，告诉操作系统，我们将使用 $HOME/test 目录作为 /bin/bash 进程的根目录
      $ chroot $HOME/test /bin/bash
      ```

    * 这时，如果执行 "ls /"，就会看到，它返回的都是 $HOME/test 目录下面的内容，而不是宿主机的内容

    * 更重要的是，对于被 chroot 的进程来说，它并不会感受到自己的根目录已经被“修改”成 $HOME/test 了

    * **实际上，Mount Namespace 正是基于对 chroot 的不断改良才被发明出来的，它也是 Linux 操作系统里的第一个 Namespace**

    * 当然，为了能够让容器的这个根目录看起来更“真实”，我们一般会在这个容器的根目录下挂载一个完整操作系统的文件系统，比如 Ubuntu16.04 的 ISO。这样，在容器启动之后，我们在容器里通过执行 "ls /" 查看根目录下的内容，就是 Ubuntu 16.04 的所有目录和文件

    * **而这个挂载在容器根目录上、用来为容器进程提供隔离后执行环境的文件系统，就是所谓的“容器镜像”。它还有一个更为专业的名字，叫作：rootfs（根文件系统）**

    * 所以，一个最常见的 rootfs，或者说容器镜像，会包括如下所示的一些目录和文件，比如 /bin，/etc，/proc 等等

      ```shell
      $ ls /
      bin dev etc home lib lib64 mnt opt proc root run sbin sys tmp usr var
      ```

    * 进入容器之后执行的 /bin/bash，就是 /bin 目录下的可执行文件，与宿主机的 /bin/bash 完全不同

    * **需要明确的是，rootfs 只是一个操作系统所包含的文件、配置和目录，并不包括操作系统内核。在 Linux 操作系统中，这两部分是分开存放的，操作系统只有在开机启动时才会加载指定版本的内核镜像**

    * 所以说，rootfs 只包括了操作系统的“躯壳”，并没有包括操作系统的“灵魂”

    * 实际上，同一台机器上的所有容器，都共享宿主机操作系统的内核

      * 如果你的应用程序需要配置内核参数、加载额外的内核模块，以及跟内核进行直接的交互，你就需要注意了：这些操作和依赖的对象，都是宿主机操作系统的内核，它对于该机器上的所有容器来说是一个“全局变量”，牵一发而动全身
      * 这也是容器相比于虚拟机的主要缺陷之一：毕竟后者不仅有模拟出来的硬件机器充当沙盒，而且每个沙盒里还运行着一个完整的 Guest OS 给应用随便折腾

    * 不过，**正是由于 rootfs 的存在，容器才有了一个被反复宣传至今的重要特性：一致性**

      * **由于 rootfs 里打包的不只是应用，而是整个操作系统的文件和目录，也就意味着，应用以及它运行所需要的所有依赖，都被封装在了一起**

      * **对一个应用来说，操作系统本身才是它运行所需要的最完整的“依赖库”**

      * 有了容器镜像“打包操作系统”的能力，这个最基础的依赖环境也终于变成了应用沙盒的一部分。这就赋予了容器所谓的一致性：无论在本地、云端，还是在一台任何地方的机器上，用户只需要解压打包好的容器镜像，那么这个应用运行所需要的完整的执行环境就被重现出来了

      * **这种深入到操作系统级别的运行环境一致性，打通了应用在本地开发和远端执行环境之间难以逾越的鸿沟**

      * 难道我每开发一个应用，或者升级一下现有的应用，都要重复制作一次 rootfs 吗？

        * 既然这些修改都基于一个旧的 rootfs，能不能以增量的方式去做这些修改呢？这样做的好处是，所有人都只需要维护相对于 base rootfs 修改的增量内容，而不是每次修改都制造一个“fork”

        * Docker 在镜像的设计中，引入了层（layer）的概念。也就是说，用户制作镜像的每一步操作，都会生成一个层，也就是一个增量 rootfs

        * 用到了一种叫作联合文件系统（Union File System）的能力

          * Union File System 也叫 UnionFS，最主要的功能是将多个不同位置的目录联合挂载（union mount）到同一个目录下

            ```shell
            # 比如，我现在有两个目录 A 和 B，它们分别有两个文件：
            $ tree
            .
            ├── A
            │  ├── a
            │  └── x
            └── B
              ├── b
              └── x
            # 使用联合挂载的方式，将这两个目录挂载到一个公共的目录 C 上
            $ mkdir C
            $ mount -t aufs -o dirs=./A:./B none ./C
            # 这时，我再查看目录 C 的内容，就能看到目录 A 和 B 下的文件被合并到了一起
            $ tree ./C
            ./C
            ├── a
            ├── b
            └── x
            #  x 文件只有一份
            # 如果你在目录 C 里对 a、b、x 文件做修改，这些修改也会在对应的目录 A、B 中生效
            ```

      * **容器的 rootfs** **三部分组成：**
        * **第一部分，只读层**
          * 都以增量的方式分别包含了操作系统的一部分
        * **第二部分，可读写层**
          * 在没有写入文件之前，这个目录是空的。而一旦在容器里做了写操作，你修改产生的内容就会以增量的方式出现在这个层中
          * 为了实现这样的删除操作，AuFS 会在可读写层创建一个 whiteout 文件，把只读层里的文件“遮挡”起来
          * 最上面这个可读写层的作用，就是专门用来存放你修改 rootfs 后产生的增量，无论是增、删、改，都发生在这里。而当我们使用完了这个被修改过的容器之后，还可以使用 docker commit 和 push 指令，保存这个被修改过的可读写层，并上传到 Docker Hub 上，供其他人使用；而与此同时，原先的只读层里的内容则不会有任何变化。这，就是增量 rootfs 的好处
        * **第三部分，Init 层**
          * 它是一个以“-init”结尾的层，夹在只读层和读写层之间
          * Init 层是 Docker 项目单独生成的一个内部层，专门用来存放 /etc/hosts、/etc/resolv.conf 等信息
          * 需要这样一层的原因是，这些文件本来属于只读的 Ubuntu 镜像的一部分，但是用户往往需要在启动容器时写入一些指定的值比如 hostname，所以就需要在可读写层对它们进行修改
          * 可是，这些修改往往只对当前的容器有效，我们并不希望执行 docker commit 时，把这些信息连同可读写层一起提交掉
          * 所以，Docker 做法是，在修改了这些文件之后，以一个单独的层挂载了出来。而用户执行 docker commit 只会提交可读写层，所以是不包含这些内容的

  * 对 Docker 项目来说，它最核心的原理实际上就是为待创建的用户进程：

    * 启用 Linux Namespace 配置
    * 设置指定的 Cgroups 参数
    * 切换进程的根目录（Change Root）

  * 这样，一个完整的容器就诞生了。不过，Docker 项目在最后一步的切换上会优先使用 pivot_root 系统调用，如果系统不支持，才会使用 chroot

*  Linux 容器文件系统的实现方式。而这种机制，正是我们经常提到的容器镜像，也叫作：rootfs。它只是一个操作系统的所有文件和目录，并不包含内核，最多也就几百兆。而相比之下，传统虚拟机的镜像大多是一个磁盘的“快照”，磁盘有多大，镜像就至少有多大

* 通过结合使用 Mount Namespace 和 rootfs，容器就能够为进程构建出一个完善的文件系统隔离环境。当然，这个功能的实现还必须感谢 chroot 和 pivot_root 这两个系统调用切换进程根目录的能力

* 而在 rootfs 的基础上，Docker 公司创新性地提出了使用多个增量 rootfs 联合挂载一个完整 rootfs 的方案，这就是容器镜像中“层”的概念

* 通过“分层镜像”的设计，以 Docker 镜像为核心，来自不同公司、不同团队的技术人员被紧密地联系在了一起。而且，由于容器镜像的操作是增量式的，这样每次镜像拉取、推送的内容，比原本多个完整的操作系统的大小要小得多；而共享层的存在，可以使得所有这些容器镜像需要的总空间，也比每个镜像的总和要小。这样就使得基于容器镜像的团队协作，要比基于动则几个 GB 的虚拟机磁盘镜像的协作要敏捷得多

* 更重要的是，一旦这个镜像被发布，那么你在全世界的任何一个地方下载这个镜像，得到的内容都完全一致，可以完全复现这个镜像制作者当初的完整环境。这，就是容器技术“强一致性”的重要体现

  ![k8s技能图谱](/Users/dingyuanjie/Downloads/k8s技能图谱.jpeg)

*  Docker 容器的本质

  * 用 Docker 部署一个用 Python 编写的 Web 应用

    * 使用 Flask 框架启动了一个 Web 服务器，而它唯一的功能是：如果当前环境中有“NAME”这个环境变量，就把它打印在“Hello”后，否则就打印“Hello world”，最后再打印出当前环境的 hostname

    * 这个应用的依赖，则被定义在了同目录下的 requirements.txt 文件里，内容如下所示：

      * ```shell
        $ ls
        Dockerfile  app.py   requirements.txt
        $ cat requirements.txt
        Flask
        ```

    ```python
    # app.py
    from flask import Flask
    import socket
    import os
     
    app = Flask(__name__)
     
    @app.route('/')
    def hello():
        html = "<h3>Hello {name}!</h3>" \
               "<b>Hostname:</b> {hostname}<br/>"           
        return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname())
        
    if __name__ == "__main__":
        app.run(host='0.0.0.0', port=80)
    ```

    * **将这样一个应用容器化的第一步，是制作容器镜像**

      ```dockerfile
      # Dockerfile
      # 使用官方提供的 Python 开发镜像作为基础镜像
      FROM python:2.7-slim
      # 将工作目录切换为 /app
      WORKDIR /app
      # 将当前目录下的所有内容复制到 /app 下
      ADD . /app
      # RUN 原语就是在容器里执行 shell 命令的意思
      # 使用 pip 命令安装这个应用所需要的依赖
      RUN pip install --trusted-host pypi.python.org -r requirements.txt
      # 允许外界访问容器的 80 端口
      EXPOSE 80
      # 设置环境变量
      ENV NAME World
      # 设置容器进程为：python app.py，即：这个 Python 应用的启动命令
      # Dockerfile 指定 python app.py 为这个容器的进程。这里，app.py 的实际路径是 /app/app.py。所以，CMD [“python”, “app.py”] 等价于 "docker run python app.py"
      CMD ["python", "app.py"]
      ```

      * **Dockerfile 的设计思想**是使用一些标准的原语（即大写高亮的词语），描述我们所要构建的 Docker 镜像。并且这些原语，都是按顺序处理的
      * 使用 Dockerfile 时，可能还会看到一个叫作 ENTRYPOINT 的原语。实际上，它和 CMD 都是 Docker 容器进程启动所必需的参数，完整执行格式是：“ENTRYPOINT CMD”
      * 但是，默认情况下，Docker 会为你提供一个隐含的 ENTRYPOINT，即：/bin/sh -c。所以，在不指定 ENTRYPOINT 时，比如在我们这个例子里，实际上运行在容器里的完整进程是：/bin/sh -c “python app.py”，即 CMD 的内容就是 ENTRYPOINT 的参数
        * 基于以上原因，**后面会统一称 Docker 容器的启动进程为 ENTRYPOINT，而不是 CMD**

    * 让 Docker 制作这个镜像了，在当前目录执行

      * **需要注意的是，Dockerfile 中的每个原语执行后，都会生成一个对应的镜像层**。即使原语本身并没有明显地修改文件的操作（比如，ENV 原语），它对应的层也会存在。只不过在外界看来，这个层是空的

      ```shell
      # -t 的作用是给这个镜像加一个 Tag，即：起一个好听的名字
      # docker build 会自动加载当前目录下的 Dockerfile 文件，然后按照顺序，执行文件中的原语。而这个过程，实际上可以等同于 Docker 使用基础镜像启动了一个容器，然后在容器中依次执行 Dockerfile 中的原语
      docker build -t helloworld .
      
      docker image ls
      # 容器内的 80 端口映射在宿主机的 4000 端口上
      # curl http://localhost:4000
      docker run -p 4000:80 helloworld
      # $ docker run -p 4000:80 helloworld python app.py
      docker ps
      ```

    * 容器的镜像上传到 DockerHub 上分享给更多的人

      * 首先，**注册一个 Docker Hub 账号，然后使用 docker login 命令登录**

      * **用 docker tag 命令给容器镜像起一个完整的名字**：

        * `docker tag helloworld geektime/helloworld:v1`
        * 其中，geektime 是 Docker Hub 上的用户名，它的“学名”叫镜像仓库（Repository）；“/”后面的 helloworld 是这个镜像的名字，而“v1”则是给这个镜像分配的版本号

      * 执行**docker push**，就可以把这个镜像上传到 Docker Hub 上

        * `docker push geektime/helloworld:v1`

      * 还可以使用 docker commit 指令，把一个正在运行的容器，直接提交为一个镜像。一般来说，需要这么操作原因是：这个容器运行起来后，我又在里面做了一些操作，并且要把操作结果保存到镜像里

        ```shell
        $ docker exec -it 4ddf4638572d /bin/sh
        # 在容器内部新建了一个文件
        root@4ddf4638572d:/app# touch test.txt
        root@4ddf4638572d:/app# exit
         
        # 将这个新建的文件提交到镜像中保存
        $ docker commit 4ddf4638572d geektime/helloworld:v2
        $ docker push geektime/helloworld:v2
        ```

        * docker commit，实际上就是在容器运行起来后，把最上层的“可读写层”，加上原先容器镜像的只读层，打包组成了一个新的镜像。当然，下面这些只读层在宿主机上是共享的，不会占用额外的空间
        * 而由于使用了联合文件系统，你在容器里对镜像 rootfs 所做的任何修改，都会被操作系统先复制到这个可读写层，然后再修改。这就是所谓的：Copy-on-Write
        * 而正如前所说，Init 层的存在，就是为了避免你执行 docker commit 时，把 Docker 自己对 /etc/hosts 等文件做的修改，也一起提交掉

    * 进入容器内

      ```shell
      # 当前正在运行的 Docker 容器的进程号（PID）是 25686
      $ docker inspect --format '{{ .State.Pid }}'  4ddf4638572d
      25686
      # 可以通过查看宿主机的 proc 文件，看到这个 25686 进程的所有 Namespace 对应的文件
      $ ls -l  /proc/25686/ns
      total 0
      lrwxrwxrwx 1 root root 0 Aug 13 14:05 cgroup -> cgroup:[4026531835]
      lrwxrwxrwx 1 root root 0 Aug 13 14:05 ipc -> ipc:[4026532278]
      lrwxrwxrwx 1 root root 0 Aug 13 14:05 mnt -> mnt:[4026532276]
      lrwxrwxrwx 1 root root 0 Aug 13 14:05 net -> net:[4026532281]
      lrwxrwxrwx 1 root root 0 Aug 13 14:05 pid -> pid:[4026532279]
      lrwxrwxrwx 1 root root 0 Aug 13 14:05 pid_for_children -> pid:[4026532279]
      lrwxrwxrwx 1 root root 0 Aug 13 14:05 user -> user:[4026531837]
      lrwxrwxrwx 1 root root 0 Aug 13 14:05 uts -> uts:[4026532277]
      # 一个进程的每种 Linux Namespace，都在它对应的 /proc/[进程号]/ns 下有一个对应的虚拟文件，并且链接到一个真实的 Namespace 文件上
      ```

      * **一个进程，可以选择加入到某个进程已有的 Namespace 当中，从而达到“进入”这个进程所在容器的目的，这正是 docker exec 的实现原理**

        * 而这个操作所依赖的，乃是一个名叫 setns() 的 Linux 系统调用

        * 一旦一个进程加入到了另一个 Namespace 当中，在宿主机的 Namespace 文件上，也会有所体现

        * 在宿主机上，可以用 ps 指令找到这个 set_ns 程序执行的 /bin/bash 进程，其真实的 PID 是 28499：

          ```shell
          # 在宿主机上
          ps aux | grep /bin/bash
          root     28499  0.0  0.0 19944  3612 pts/0    S    14:15   0:00 /bin/bash
          # 查看一下这个 PID=28499 的进程的 Namespace，就会发现这样一个事实
          $ ls -l /proc/28499/ns/net
          lrwxrwxrwx 1 root root 0 Aug 13 14:18 /proc/28499/ns/net -> net:[4026532281]
          $ ls -l  /proc/25686/ns/net
          lrwxrwxrwx 1 root root 0 Aug 13 14:05 /proc/25686/ns/net -> net:[4026532281]
          # 在 /proc/[PID]/ns/net 目录下，这个 PID=28499 进程，与我们前面的 Docker 容器进程（PID=25686）指向的 Network Namespace 文件完全一样。这说明这两个进程，共享了这个名叫 net:[4026532281] 的 Network Namespace
          ```

    * Docker 还专门提供了一个参数，可以让你启动一个容器并“加入”到另一个容器的 Network Namespace 里，这个参数就是 -net

      ```shell
      docker run -it --net container:4ddf4638572d busybox ifconfig
      ```

      * 新启动的这个容器，就会直接加入到 ID=4ddf4638572d 的容器，也就是我们前面的创建的 Python 应用容器（PID=25686）的 Network Namespace 中。所以，这里 ifconfig 返回的网卡信息，跟我前面那个小程序返回的结果一模一样
      * 而如果指定–net=host，就意味着这个容器不会为进程启用 Network Namespace。这就意味着，这个容器拆除了 Network Namespace 的“隔离墙”，所以，它会和宿主机上的其他普通进程一样，直接共享宿主机的网络栈。这就为容器直接操作和使用宿主机网络提供了一个渠道

    * Volume（数据卷）

      * 容器里进程新建的文件，怎么才能让宿主机获取到？
      * 宿主机上的文件和目录，怎么才能让容器里的进程访问到？
      * 这正是 Docker Volume 要解决的问题：
        * **Volume 机制，允许你将宿主机上指定的目录或者文件，挂载到容器里面进行读取和修改操作**

      ```shell
      # 在 Docker 项目里，它支持两种 Volume 声明方式，可以把宿主机目录挂载进容器的 /test 目录当中
      # 本质，实际上是相同的：都是把一个宿主机的目录挂载进了容器的 /test 目录
      # 只不过，在第一种情况下，由于你并没有显示声明宿主机目录，那么 Docker 就会默认在宿主机上创建一个临时目录 /var/lib/docker/volumes/[VOLUME_ID]/_data，然后把它挂载到容器的 /test 目录上。而在第二种情况下，Docker 就直接把宿主机的 /home 目录挂载到容器的 /test 目录上
      $ docker run -v /test ...
      $ docker run -v /home:/test ...
      ```

      * 当容器进程被创建之后，尽管开启了 Mount Namespace，但是在它执行 chroot（或者 pivot_root）之前，容器进程一直可以看到宿主机上的整个文件系统
      * 而宿主机上的文件系统，也自然包括了我们要使用的容器镜像。这个镜像的各个层，保存在 /var/lib/docker/aufs/diff 目录下，在容器进程启动后，它们会被联合挂载在 /var/lib/docker/aufs/mnt/ 目录中，这样容器所需的 rootfs 就准备好了
      * 所以，我们只需要在 rootfs 准备好之后，在执行 chroot 之前，把 Volume 指定的宿主机目录（比如 /home 目录），挂载到指定的容器目录（比如 /test 目录）在宿主机上对应的目录（即 /var/lib/docker/aufs/mnt/[可读写层 ID]/test）上，这个 Volume 的挂载工作就完成了
      * 更重要的是，由于执行这个挂载操作时，“容器进程”已经创建了，也就意味着此时 Mount Namespace 已经开启了。所以，这个挂载事件只在这个容器里可见。你在宿主机上，是看不见容器内部的这个挂载点的。这就**保证了容器的隔离性不会被 Volume 打破**
      * 注意：这里提到的 " 容器进程 "，是 Docker 创建的一个容器初始化进程 (dockerinit)，而不是应用进程 (ENTRYPOINT + CMD)。dockerinit 会负责完成根目录的准备、挂载设备和目录、配置 hostname 等一系列需要在容器内进行的初始化操作。最后，它通过 execv() 系统调用，让应用进程取代自己，成为容器里的 PID=1 的进程
      * 而这里要使用到的挂载技术，就是 Linux 的**绑定挂载（bind mount）机制**。它的主要作用就是，允许你将一个目录或者文件，而不是整个设备，挂载到一个指定的目录上。并且，这时你在该挂载点上进行的任何操作，只是发生在被挂载的目录或者文件上，而原挂载点的内容则会被隐藏起来且不受影响
      * 绑定挂载实际上是一个 inode 替换的过程。在 Linux 操作系统中，inode 可以理解为存放文件内容的“对象”，而 dentry，也叫目录项，就是访问这个 inode 所使用的“指针”
      * mount --bind /home /test，会将 /home 挂载到 /test 上。其实相当于将 /test 的 dentry，重定向到了 /home 的 inode。这样当我们修改 /test 目录时，实际修改的是 /home 目录的 inode。这也就是为何，一旦执行 umount 命令，/test 目录原先的内容就会恢复：因为修改真正发生在的，是 /home 目录里
      * **所以，在一个正确的时机，进行一次绑定挂载，Docker 就可以成功地将一个宿主机上的目录或文件，不动声色地挂载到容器中**
      * 这样，进程在容器里对这个 /test 目录进行的所有操作，都实际发生在宿主机的对应目录（比如，/home，或者 /var/lib/docker/volumes/[VOLUME_ID]/_data）里，而不会影响容器镜像的内容
      * 那么，这个 /test 目录里的内容，既然挂载在容器 rootfs 的可读写层，它会不会被 docker commit 提交掉呢？
        * 也不会
        * 容器的镜像操作，比如 docker commit，都是发生在宿主机空间的。而由于 Mount Namespace 的隔离作用，宿主机并不知道这个绑定挂载的存在。所以，在宿主机看来，容器中可读写层的 /test 目录（/var/lib/docker/aufs/mnt/[可读写层 ID]/test），**始终是空的**
        * 不过，由于 Docker 一开始还是要创建 /test 这个目录作为挂载点，所以执行了 docker commit 之后，你会发现新产生的镜像里，会多出来一个空的 /test 目录。毕竟，新建目录操作，又不是挂载操作，Mount Namespace 对它可起不到“障眼法”的作用

      ```shell
      # 首先，启动一个 helloworld 容器，给它声明一个 Volume，挂载在容器里的 /test 目录上：
      $ docker run -d -v /test helloworld
      cf53b766fa6f
      # 容器启动之后，我们来查看一下这个 Volume 的 ID：
      $ docker volume ls
      DRIVER              VOLUME NAME
      local               cb1c2f7221fa9b0971cc35f68aa1034824755ac44a034c0c0a1dd318838d3a6d
      # 然后，使用这个 ID，可以找到它在 Docker 工作目录下的 volumes 路径
      $ ls /var/lib/docker/volumes/cb1c2f7221fa/_data/
      # 这个 _data 文件夹，就是这个容器的 Volume 在宿主机上对应的临时目录了
      # 在容器的 Volume 里，添加一个文件 text.txt
      $ docker exec -it cf53b766fa6f /bin/sh
      cd test/
      touch text.txt
      # 再回到宿主机，就会发现 text.txt 已经出现在了宿主机上对应的临时目录里：
      $ ls /var/lib/docker/volumes/cb1c2f7221fa/_data/
      text.txt
      # 可是，如果你在宿主机上查看该容器的可读写层，虽然可以看到这个 /test 目录，但其内容是空的
      $ ls /var/lib/docker/aufs/mnt/6780d0778b8a/test
      # 可以确认容器 Volume 里的信息，并不会被 docker commit 提交掉；但这个挂载点目录 /test 本身，则会出现在新的镜像当中
      ```

      ![下载](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/下载.png)

      * 这个容器进程“python app.py”，运行在由 Linux Namespace 和 Cgroups 构成的隔离环境里；而它运行所需要的各种文件，比如 python，app.py，以及整个操作系统文件，则由多个联合挂载在一起的 rootfs 层提供
      * 这些 rootfs 层的最下层，是来自 Docker 镜像的只读层
      * 在只读层之上，是 Docker 自己添加的 Init 层，用来存放被临时修改过的 /etc/hosts 等文件
      * 而 rootfs 的最上层是一个可读写层，它以 Copy-on-Write 的方式存放任何对只读层的修改，容器声明的 Volume 的挂载点，也出现在这一层
      * 思考题
        * 你在查看 Docker 容器的 Namespace 时，是否注意到有一个叫 cgroup 的 Namespace？它是 Linux 4.6 之后新增加的一个 Namespace，你知道它的作用吗？
        * 如果你执行 docker run -v /home:/test 的时候，容器镜像里的 /test 目录下本来就有内容的话，你会发现，在宿主机的 /home 目录下，也会出现这些内容。这是怎么回事？为什么它们没有被绑定挂载隐藏起来呢？（提示：Docker 的“copyData”功能）
        * 请尝试给这个 Python 应用加上 CPU 和 Memory 限制，然后启动它。根据我们前面介绍的 Cgroups 的知识，请你查看一下这个容器的 Cgroups 文件系统的设置，是不是跟我前面的讲解一致

      

      * 一个“容器”，实际上是一个由 Linux Namespace、Linux Cgroups 和 rootfs 三种技术构建出来的进程的隔离环境
        * 一个正在运行的 Linux 容器，其实可以被“一分为二”地看待：
          * 一组联合挂载在 /var/lib/docker/aufs/mnt 上的 rootfs，这一部分我们称为“容器镜像”（Container Image），是容器的静态视图；
          * 一个由 Namespace+Cgroups 构成的隔离环境，这一部分我们称为“容器运行时”（Container Runtime），是容器的动态视图。
        * 在整个“开发 - 测试 - 发布”的流程中，真正承载着容器信息进行传递的，是容器镜像，而不是容器运行时

      

      * 首先，Kubernetes 项目要解决的问题是什么？

        * 编排？调度？容器云？还是集群管理？

        * 实际上，这个问题到目前为止都没有固定的答案。因为在不同的发展阶段，Kubernetes 需要着重解决的问题是不同的

        * 但是，对于大多数用户来说，他们希望 Kubernetes 项目带来的体验是确定的：现在我有了应用的容器镜像，请帮我在一个给定的集群上把这个应用运行起来

        * 更进一步地说，我还希望 Kubernetes 能给我提供路由网关、水平扩展、监控、备份、灾难恢复等一系列运维能力

          * Kubernetes 项目的架构

          ![image-20200717092236316](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200717092236316.png)

          * 由 Master 和 Node 两种节点组成，而这两种角色分别对应着控制节点和计算节点
          * 其中，控制节点，即 Master 节点，由三个紧密协作的独立组件组合而成，它们分别是负责 API 服务的 kube-apiserver、负责调度的 kube-scheduler，以及负责容器编排的 kube-controller-manager。整个集群的持久化数据，则由 kube-apiserver 处理后保存在 Etcd 中
          * 而计算节点上最核心的部分，则是一个叫作 kubelet 的组件
          * **在 Kubernetes 项目中，kubelet 主要负责同容器运行时（比如 Docker 项目）打交道**。而这个交互所依赖的，是一个称作 CRI（Container Runtime Interface）的远程调用接口，这个接口定义了容器运行时的各项核心操作，比如：启动一个容器需要的所有参数
            * 这也是为何，Kubernetes 项目并不关心你部署的是什么容器运行时、使用的什么技术实现，只要你的这个容器运行时能够运行标准的容器镜像，它就可以通过实现 CRI 接入到 Kubernetes 项目当中
          * 而具体的容器运行时，比如 Docker 项目，则一般通过 OCI 这个容器运行时规范同底层的 Linux 操作系统进行交互，即：把 CRI 请求翻译成对 Linux 操作系统的调用（操作 Linux Namespace 和 Cgroups 等）
          * **此外，kubelet 还通过 gRPC 协议同一个叫作 Device Plugin 的插件进行交互**。这个插件，是 Kubernetes 项目用来管理 GPU 等宿主机物理设备的主要组件，也是基于 Kubernetes 项目进行机器学习训练、高性能作业支持等工作必须关注的功能
          * 而**kubelet 的另一个重要功能，则是调用网络插件和存储插件为容器配置网络和持久化存储**。这两个插件与 kubelet 进行交互的接口，分别是 CNI（Container Networking Interface）和 CSI（Container Storage Interface）

        * 如何编排、管理、调度用户提交的作业？

        * **从一开始，Kubernetes 项目就没有像同时期的各种“容器云”项目那样，把 Docker 作为整个架构的核心，而仅仅把它作为最底层的一个容器运行时实现**

        *  Kubernetes 项目要着重解决的问题

          * 运行在大规模集群中的各种任务之间，实际上存在着各种各样的关系。这些关系的处理，才是作业编排和管理系统最困难的地方

        * **Kubernetes 项目最主要的设计思想是，从更宏观的角度，以统一的方式来定义任务之间的各种关系，并且为将来支持更多种类的关系留有余地**

          * 在 Compose 项目中，你可以为这样的两个容器定义一个“link”，而 Docker 项目则会负责维护这个“link”关系，其具体做法是：Docker 会在 Web 容器中，将 DB 容器的 IP 地址、端口等信息以环境变量的方式注入进去，供应用进程使用，而当 DB 容器发生变化时（比如，镜像更新，被迁移到其他宿主机上等等），这些环境变量的值会由 Docker 项目自动更新。**这就是平台项目自动地处理容器间关系的典型例子**
          * 可是，如果我们现在的需求是，要求这个项目能够处理前面提到的所有类型的关系，甚至还要能够支持未来可能出现的更多种类的关系呢
          * 这时，“link”这种单独针对一种案例设计的解决方案就太过简单了。如果你做过架构方面的工作，就会深有感触：一旦要追求项目的普适性，那就一定要从顶层开始做好设计

        * Kubernetes 项目对容器间的“访问”进行了分类，首先总结出了一类非常常见的“紧密交互”的关系，即：这些应用之间需要非常频繁的交互和访问；又或者，它们会直接通过本地文件进行信息交换

        * 在常规环境下，这些应用往往会被直接部署在同一台机器上，通过 Localhost 通信，通过本地磁盘目录交换文件。而在 Kubernetes 项目中，这些容器则会被划分为一个“Pod”，Pod 里的容器共享同一个 Network Namespace、同一组数据卷，从而达到高效率交换信息的目的

        * Pod 是 Kubernetes 项目中最基础的一个对象

        * 对于另外一种更为常见的需求，比如 Web 应用与数据库之间的访问关系，Kubernetes 项目则提供了一种叫作“Service”的服务。像这样的两个应用，往往故意不部署在同一台机器上，这样即使 Web 应用所在的机器宕机了，数据库也完全不受影响。可是，我们知道，对于一个容器来说，它的 IP 地址等信息不是固定的，那么 Web 应用又怎么找到数据库容器的 Pod 呢？

          * 所以，Kubernetes 项目的做法是给 Pod 绑定一个 Service 服务，而 Service 服务声明的 IP 地址等信息是“终生不变”的。这个**Service 服务的主要作用，就是作为 Pod 的代理入口（Portal），从而代替 Pod 对外暴露一个固定的网络地址**
          * 这样，对于 Web 应用的 Pod 来说，它需要关心的就是数据库 Pod 的 Service 信息。不难想象，Service 后端真正代理的 Pod 的 IP 地址、端口等信息的自动更新、维护，则是 Kubernetes 项目的职责

      * Kubernetes 项目核心功能的“全景图”

        ![image-20200717093945864](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200717093945864.png)

        * 按照这幅图的线索，我们从容器这个最基础的概念出发，首先遇到了容器间“紧密协作”关系的难题，于是就扩展到了 Pod；有了 Pod 之后，我们希望能一次启动多个应用的实例，这样就需要 Deployment 这个 Pod 的多实例管理器；而有了这样一组相同的 Pod 后，我们又需要通过一个固定的 IP 地址和端口以负载均衡的方式访问它，于是就有了 Service
        * 可是，如果现在两个不同 Pod 之间不仅有“访问关系”，还要求在发起时加上授权信息。最典型的例子就是 Web 应用对数据库访问时需要 Credential（数据库的用户名和密码）信息。那么，在 Kubernetes 中这样的关系又如何处理呢？
        * Kubernetes 项目提供了一种叫作 Secret 的对象，它其实是一个保存在 Etcd 里的键值对数据。这样，你把 Credential 信息以 Secret 的方式存在 Etcd 里，Kubernetes 就会在你指定的 Pod（比如，Web 应用的 Pod）启动时，自动把 Secret 里的数据以 Volume 的方式挂载到容器里。这样，这个 Web 应用就可以访问数据库了

      * **除了应用与应用之间的关系外，应用运行的形态是影响“如何容器化这个应用”的第二个重要因素**

        * 为此，Kubernetes 定义了新的、基于 Pod 改进后的对象。比如 Job，用来描述一次性运行的 Pod（比如，大数据任务）；再比如 DaemonSet，用来描述每个宿主机上必须且只能运行一个副本的守护进程服务；又比如 CronJob，则用于描述定时任务等等
        * 如此种种，正是 Kubernetes 项目定义容器间关系和形态的主要方法

      * 可以看到，Kubernetes 项目并没有像其他项目那样，为每一个管理功能创建一个指令，然后在项目中实现其中的逻辑。这种做法，的确可以解决当前的问题，但是在更多的问题来临之后，往往会力不从心

      * 相比之下，在 Kubernetes 项目中，我们所推崇的使用方法是

        * 首先，通过一个“编排对象”，比如 Pod、Job、CronJob 等，来描述你试图管理的应用
        * 然后，再为它定义一些“服务对象”，比如 Service、Secret、Horizontal Pod Autoscaler（自动水平扩展器）等。这些对象，会负责具体的平台级功能

      * **这种使用方法，就是所谓的“声明式 API”。这种 API 对应的“编排对象”和“服务对象”，都是 Kubernetes 项目中的 API 对象（API Object）**

      * Kubernetes 项目如何启动一个容器化任务呢？

        * 比如，我现在已经制作好了一个 Nginx 容器镜像，希望让平台帮我启动这个镜像。并且，我要求平台帮我运行两个完全相同的 Nginx 副本，以负载均衡的方式共同对外提供服务

          ```yaml
          #  nginx-deployment.yaml
          apiVersion: apps/v1
          kind: Deployment
          metadata:
            name: nginx-deployment
            labels:
              app: nginx
          spec:
            replicas: 2
            selector:
              matchLabels:
                app: nginx
            template:
              metadata:
                labels:
                  app: nginx
              spec:
                containers:
                - name: nginx
                  image: nginx:1.7.9
                  ports:
                  - containerPort: 80
          ```

          ```python
          # 执行
          kubectl create -f nginx-deployment.yaml
          ```

  * Kubernetes 项目的架构，使用“声明式 API”来描述容器化业务和容器间关系的设计思想
    * 实际上，过去很多的集群管理项目（比如 Yarn、Mesos，以及 Swarm）所擅长的，都是把一个容器，按照某种规则，放置在某个最佳节点上运行起来。这种功能，我们称为“调度”
    * 而 Kubernetes 项目所擅长的，是按照用户的意愿和整个系统的规则，完全自动化地处理好容器之间的各种关系。**这种功能，就是我们经常听到的一个概念：编排。**
    * 所以说，Kubernetes 项目的本质，是为用户提供一个具有普遍意义的容器编排工具
    * 不过，更重要的是，Kubernetes 项目为用户提供的不仅限于一个工具。它真正的价值，乃在于提供了一套基于容器构建分布式系统的基础依赖

  

  

  

  

