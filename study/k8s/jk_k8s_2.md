*  **Kubernetes 项目简单的部署方法**

  * 独立的部署工具，名叫：[kubeadm](https://github.com/kubernetes/kubeadm)

    ```shell
    # 让用户能够通过这样两条指令完成一个 Kubernetes 集群的部署
    # 创建一个 Master 节点
    $ kubeadm init
     
    # 将一个 Node 节点加入到当前集群中
    $ kubeadm join <Master 节点的 IP 和端口 >
    ```

## kubeadm 的工作原理

* Kubernetes 的架构和它的组件，在部署时，它的每一个组件都是一个需要被执行的、单独的二进制文件

* **为什么不用容器部署 Kubernetes 呢？**

  * 只要给每个 Kubernetes 组件做一个容器镜像，然后在每台宿主机上用 docker run 指令启动这些组件容器，部署不就完成了吗？
  * 但是，**这样做会带来一个很麻烦的问题，即：如何容器化 kubelet**
  *  kubelet 是 Kubernetes 项目用来操作 Docker 等容器运行时的核心组件。可是，除了跟容器运行时打交道外，kubelet 在配置容器网络、管理容器数据卷时，都需要直接操作宿主机
  * 而如果现在 kubelet 本身就运行在一个容器里，那么直接操作宿主机就会变得很麻烦。对于网络配置来说还好，kubelet 容器可以通过不开启 Network Namespace（即 Docker 的 host network 模式）的方式，直接共享宿主机的网络栈。可是，要让 kubelet 隔着容器的 Mount Namespace 和文件系统，操作宿主机的文件系统，就有点儿困难了
    * kubelet 做的挂载操作，不能被“传播”到宿主机上
  * 到目前为止，在容器里运行 kubelet，依然没有很好的解决办法，也不推荐用容器去部署 Kubernetes 项目

* kubeadm 选择了一种妥协方案

  * 把 kubelet 直接运行在宿主机上，然后使用容器部署其他的 Kubernetes 组件

  * 使用 kubeadm 的第一步，是在机器上手动安装 kubeadm、kubelet 和 kubectl 这三个二进制文件

    ```shell
    # 只需执行
    $ apt-get install kubeadm
    # 接下来，就可以使用“kubeadm init”部署 Master 节点
    ```

    