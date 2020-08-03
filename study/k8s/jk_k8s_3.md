# 编排：控制器

* **Pod 这个看似复杂的 API 对象，实际上就是对容器的进一步抽象和封装而已**

* Pod 对象，其实就是容器的升级版。它对容器进行了组合，添加了更多的属性和字段

  * 这就好比给集装箱四面安装了吊环，使得 Kubernetes 这架“吊车”，可以更轻松地操作它
  * 而 Kubernetes 操作这些“集装箱”的逻辑，都由控制器（Controller）完成

* Deployment 这个最基本的控制器对象

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment
  spec:
    selector:
      matchLabels:
        app: nginx
    replicas: 2
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

  * 确保携带了 app=nginx 标签的 Pod 的个数，永远等于 spec.replicas 指定的个数，即 2 个

  * 究竟是 Kubernetes 项目中的哪个组件，在执行这些操作呢？

  *  kube-controller-manager 的组件就是一系列控制器的集合

    *  Kubernetes 项目的 pkg/controller 目录：

      ```shell
      $ cd kubernetes/pkg/controller/
      # 这个目录下面的每一个控制器，都以独有的方式负责某种编排功能
      $ ls -d */              
      deployment/             job/                    podautoscaler/          
      cloud/                  disruption/             namespace/              
      replicaset/             serviceaccount/         volume/
      cronjob/                garbagecollector/       nodelifecycle/          replication/            statefulset/            daemon/
      ...
      ```

    * 都遵循 Kubernetes 项目中的一个通用编排模式，即：控制循环（control loop）

      ```java
      for {
        实际状态 := 获取集群中对象 X 的实际状态（Actual State）
        期望状态 := 获取集群中对象 X 的期望状态（Desired State）
        if 实际状态 == 期望状态{
          什么都不做
        } else {
          执行编排动作，将实际状态调整为期望状态
        }
      }
      ```

      * **在具体实现中，实际状态往往来自于 Kubernetes 集群本身**
        * 比如，kubelet 通过心跳汇报的容器状态和节点状态，或者监控系统中保存的应用监控数据，或者控制器主动收集的它自己感兴趣的信息，这些都是常见的实际状态的来源
      * **而期望状态，一般来自于用户提交的 YAML 文件**
        * 比如，Deployment 对象中 Replicas 字段的值。很明显，这些信息往往都保存在 Etcd 中

* Deployment 对控制器模型的实现

  1. Deployment 控制器从 Etcd 中获取到所有携带了“app: nginx”标签的 Pod，然后统计它们的数量，这就是实际状态
  2. Deployment 对象的 Replicas 字段的值就是期望状态
  3. Deployment 控制器将两个状态做比较，然后根据比较结果，确定是创建 Pod，还是删除已有的 Pod

* 可以看到，一个 Kubernetes 对象的主要编排逻辑，实际上是在第三步的“对比”阶段完成的

  * 这个操作，通常被叫作调谐（Reconcile）。这个调谐的过程，则被称作“Reconcile Loop”（调谐循环）或者“Sync Loop”（同步循环）
  * 其实指的都是同一个东西：控制循环
  * 而调谐的最终结果，往往都是对被控制对象的某种写操作。比如，增加 Pod，删除已有的 Pod，或者更新 Pod 的某个字段。**这也是 Kubernetes 项目“面向 API 对象编程”的一个直观体现**

* 像 Deployment 这种控制器的设计原理，就是用一种对象管理另一种对象”的“艺术”

  * 其中，这个控制器对象本身，负责定义被管理对象的期望状态。比如，Deployment 里的 replicas=2 这个字段

  * 而被控制对象的定义，则来自于一个“模板”。比如，Deployment 里的 template 字段

  * 可以看到，Deployment 这个 template 字段里的内容，跟一个标准的 Pod 对象的 API 定义，丝毫不差。而所有被这个 Deployment 管理的 Pod 实例，其实都是根据这个 template 字段的内容创建出来的

  * 像 Deployment 定义的 template 字段，在 Kubernetes 项目中有一个专有的名字，叫作 PodTemplate（Pod 模板）

  * 大多数控制器，都会使用 PodTemplate 来统一定义它所要管理的 Pod

    ![image-20200728093921940](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200728093921940.png)

    * **类似 Deployment 这样的一个控制器，实际上都是由上半部分的控制器定义（包括期望状态），加上下半部分的被控制对象的模板组成的**
    * 这就是为什么，在所有 API 对象的 Metadata 里，都有一个字段叫作 ownerReference，用于保存当前这个 API 对象的拥有者（Owner）的信息

* 不同类型的容器编排功能，比如 StatefulSet、DaemonSet 等等，它们无一例外地都有这样一个甚至多个控制器的存在，并遵循控制循环（control loop）的流程，完成各自的编排逻辑
* 实际上，跟 Deployment 相似，这些控制循环最后的执行结果，要么就是创建、更新一些 Pod（或者其他的 API 对象、资源），要么就是删除一些已经存在的 Pod（或者其他的 API 对象、资源）
* 但也正是在这个统一的编排框架下，不同的控制器可以在具体执行过程中，设计不同的业务逻辑，从而达到不同的编排效果
* Kubernetes 使用的这个“控制器模式”，跟我们平常所说的“事件驱动”，有什么区别和联系？
  * “事件驱动”，对于控制器来说是被动，只要触发事件则执行。“控制器模式”，对于控制器来说是主动的，自身在不断地获取信息。
  * 事件往往是一次性的，如果操作失败比较难处理，但是控制器是循环一直在尝试的，更符合kubernetes申明式API，最终达到与申明一致

# 作业副本与水平扩展

## Deployment

* 是Kubernetes 里第一个控制器模式的完整实现，实现了Pod 的“水平扩展 / 收缩”（horizontal scaling out/in）

  * 例如，如果更新了 Deployment 的 Pod 模板（比如，修改了容器的镜像），那么 Deployment 就需要遵循一种叫作“滚动更新”（rolling update）的方式，来升级现有的容器

  * 而这个能力的实现，依赖的是 Kubernetes 项目中的一个非常重要的概念（API 对象）：ReplicaSet

    ```yaml
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: nginx-set
      labels:
        app: nginx
    spec:
      replicas: 3
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
    ```

    * **一个 ReplicaSet 对象，其实就是由副本数目的定义和一个 Pod 模板组成的**。不难发现，它的定义其实是 Deployment 的一个子集
    * **更重要的是，Deployment 控制器实际操纵的，正是这样的 ReplicaSet 对象，而不是 Pod 对象**
    * 对于一个 Deployment 所管理的 Pod，它的 ownerReference 是ReplicaSet

* Deployment，与 ReplicaSet，以及 Pod 的关系

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment
    labels:
      app: nginx
  spec:
    replicas: 3
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

  ![image-20200728095004802](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200728095004802.png)

  * 实际上是一种“层层控制”的关系

  * ReplicaSet 负责通过“控制器模式”，保证系统中 Pod 的个数永远等于指定的个数。这也正是 Deployment 只允许容器的 restartPolicy=Always 的主要原因：只有在容器能保证自己始终是 Running 状态的前提下，ReplicaSet 调整 Pod 的个数才有意义

  * 而在此基础上，Deployment 同样通过“控制器模式”，来操作 ReplicaSet 的个数和属性，进而实现“水平扩展 / 收缩”和“滚动更新”这两个编排动作

  * 其中，“水平扩展 / 收缩”非常容易实现，Deployment Controller 只需要修改它所控制的 ReplicaSet 的 Pod 副本个数就可以了

    * 比如，把这个值从 3 改成 4，那么 Deployment 所对应的 ReplicaSet，就会根据修改后的值自动创建一个新的 Pod。这就是“水平扩展”了；“水平收缩”则反之

    * 想要执行这个操作的指令也非常简单，就是 kubectl scale

      ```shell
      $ kubectl scale deployment nginx-deployment --replicas=4
      deployment.apps/nginx-deployment scaled
      ```

  * 滚动更新的过程

    ```shell
    # 创建这个 nginx-deployment
    # record 参数作用，是记录下每次操作所执行的命令，以方便后面查看
    kubectl create -f nginx-deployment.yaml --record
    # 检查一下 nginx-deployment 创建后的状态信息
    $ kubectl get deployments
    NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3         0         0            0           1s
    ```

    1. DESIRED：用户期望的 Pod 副本个数（spec.replicas 的值）
    2. CURRENT：当前处于 Running 状态的 Pod 的个数
    3. UP-TO-DATE：当前处于最新版本的 Pod 的个数，所谓最新版本指的是 Pod 的 Spec 部分与 Deployment 里 Pod 模板里定义的完全一致
    4. AVAILABLE：当前已经可用的 Pod 的个数，即：既是 Running 状态，又是最新版本，并且已经处于 Ready（健康检查正确）状态的 Pod 的个数

  * 实时查看 Deployment 对象的状态变化的指令 kubectl rollout status

    ```shell
    $ kubectl rollout status deployment/nginx-deployment
    Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
    deployment.apps/nginx-deployment successfully rolled out
    ```

  * 查看一下这个 Deployment 所控制的 ReplicaSet

    ```shell
    $ kubectl get rs
    NAME                          DESIRED   CURRENT   READY   AGE
    nginx-deployment-3167673210   3         3         3       20s
    ```
    * 在提交了一个 Deployment 对象后，Deployment Controller 就会立即创建一个 Pod 副本个数为 3 的 ReplicaSet。这个 ReplicaSet 的名字，则是由 Deployment 的名字和一个随机字符串共同组成
    * 这个随机字符串叫作 pod-template-hash。ReplicaSet 会把这个随机字符串加在它所控制的所有 Pod 的标签里，从而保证这些 Pod 不会与集群里的其他 Pod 混淆
    * 而 ReplicaSet 的 DESIRED、CURRENT 和 READY 字段的含义，和 Deployment 中是一致的。所以，**相比之下，Deployment 只是在 ReplicaSet 的基础上，添加了 UP-TO-DATE 这个跟版本有关的状态字段**

  * 这个时候，如果修改了 Deployment 的 Pod 模板，“滚动更新”就会被自动触发

    * 修改 Deployment 有很多方法，可以直接使用 kubectl edit 指令编辑 Etcd 里的 API 对象

      ```shell
      $ kubectl edit deployment/nginx-deployment
      ... 
          spec:
            containers:
            - name: nginx
              image: nginx:1.9.1 # 1.7.9 -> 1.9.1
              ports:
              - containerPort: 80
      ...
      deployment.extensions/nginx-deployment edited
      ```

    * kubectl edit 指令，会直接打开 nginx-deployment 的 API 对象。然后，就可以修改这里的 Pod 模板部分了。比如，在这里，将 nginx 镜像的版本升级到了 1.9.1

      * kubectl edit 并不神秘，它不过是把 API 对象的内容下载到了本地文件，让你修改完成后再提交上去

    * kubectl edit 指令编辑完成后，保存退出，Kubernetes 就会立刻触发“滚动更新”的过程

      * 可通过 kubectl rollout status 指令查看 nginx-deployment 的状态变化

        ```shell
        $ kubectl rollout status deployment/nginx-deployment
        Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
        deployment.extensions/nginx-deployment successfully rolled out
        ```

    * 可以通过查看 Deployment 的 Events，看到这个“滚动更新”的流程

      ```shell
      $ kubectl describe deployment nginx-deployment
      ...
      Events:
        Type    Reason             Age   From                   Message
        ----    ------             ----  ----                   -------
      ...
        Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set nginx-deployment-1764197365 to 1
        Normal  ScalingReplicaSet  22s   deployment-controller  Scaled down replica set nginx-deployment-3167673210 to 2
        Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deployment-1764197365 to 2
        Normal  ScalingReplicaSet  19s   deployment-controller  Scaled down replica set nginx-deployment-3167673210 to 1
        Normal  ScalingReplicaSet  19s   deployment-controller  Scaled up replica set nginx-deployment-1764197365 to 3
        Normal  ScalingReplicaSet  14s   deployment-controller  Scaled down replica set nginx-deployment-3167673210 to 0
      ```

      * 首先，当修改了 Deployment 里的 Pod 定义之后，Deployment Controller 会使用这个修改后的 Pod 模板，创建一个新的 ReplicaSet（hash=1764197365），这个新的 ReplicaSet 的初始 Pod 副本数是：0

      * 然后，在 Age=24 s 的位置，Deployment Controller 开始将这个新的 ReplicaSet 所控制的 Pod 副本数从 0 个变成 1 个，即：“水平扩展”出一个副本

      * 紧接着，在 Age=22 s 的位置，Deployment Controller 又将旧的 ReplicaSet（hash=3167673210）所控制的旧 Pod 副本数减少一个，即：“水平收缩”成两个副本

      * 如此交替进行，新 ReplicaSet 管理的 Pod 副本数，从 0 个变成 1 个，再变成 2 个，最后变成 3 个。而旧的 ReplicaSet 管理的 Pod 副本数则从 3 个变成 2 个，再变成 1 个，最后变成 0 个。这样，就完成了这一组 Pod 的版本升级过程

      * 像这样，**将一个集群中正在运行的多个 Pod 版本，交替地逐一升级的过程，就是“滚动更新”**

      * 在这个“滚动更新”过程完成之后，可以查看一下新、旧两个 ReplicaSet 的最终状态：

        ```shell
        $ kubectl get rs
        NAME                          DESIRED   CURRENT   READY   AGE
        nginx-deployment-1764197365   3         3         3       6s
        nginx-deployment-3167673210   0         0         0       30s
        ```

    * **这种“滚动更新”的好处是显而易见的**

      * 比如，在升级刚开始的时候，集群里只有 1 个新版本的 Pod。如果这时，新版本 Pod 有问题启动不起来，那么“滚动更新”就会停止，从而允许开发和运维人员介入。而在这个过程中，由于应用本身还有两个旧版本的 Pod 在线，所以服务并不会受到太大的影响

      * 当然，这也就要求一定要使用 Pod 的 Health Check 机制检查应用的运行状态，而不是简单地依赖于容器的 Running 状态。要不然的话，虽然容器已经变成 Running 了，但服务很有可能尚未启动，“滚动更新”的效果也就达不到了

      * 而为了进一步保证服务的连续性，Deployment Controller 还会确保，在任何时间窗口内，只有指定比例的 Pod 处于离线状态。同时，它也会确保，在任何时间窗口内，只有指定比例的新 Pod 被创建出来。这两个比例的值都是可以配置的，默认都是 DESIRED 值的 25%

      * 所以，在上面这个 Deployment 的例子中，它有 3 个 Pod 副本，那么控制器在“滚动更新”的过程中永远都会确保至少有 2 个 Pod 处于可用状态，至多只有 4 个 Pod 同时存在于集群中。这个策略，是 Deployment 对象的一个字段，名叫 RollingUpdateStrategy

        ```yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: nginx-deployment
          labels:
            app: nginx
        spec:
        ...
          strategy:
            type: RollingUpdate
            rollingUpdate:
              maxSurge: 1
              maxUnavailable: 1
        ```

        * maxSurge 指定的是除了 DESIRED 数量之外，在一次“滚动”中，Deployment 控制器还可以创建多少个新 Pod
        * 而 maxUnavailable 指的是，在一次“滚动”中，Deployment 控制器可以删除多少个旧 Pod
        * 这两个配置还可以用百分比形式来表示，比如：maxUnavailable=50%，指的是最多可以一次删除“50%*DESIRED 数量”个 Pod

* 扩展一下 Deployment、ReplicaSet 和 Pod 的关系

  ![image-20200728101655910](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200728101655910.png)

  * Deployment 的控制器，实际上控制的是 ReplicaSet 的数目，以及每个 ReplicaSet 的属性
  * 而一个应用的版本，对应的正是一个 ReplicaSet；这个版本应用的 Pod 数量，则由 ReplicaSet 通过它自己的控制器（ReplicaSet Controller）来保证
  * 通过这样的多个 ReplicaSet 对象，Kubernetes 项目就实现了对多个“应用版本”的描述

* Deployment 对应用进行版本控制的具体原理

  * 使用一个叫**kubectl set image**的指令，直接修改 nginx-deployment 所使用的镜像

    * 这个命令的好处就是，可以不用像 kubectl edit 那样需要打开编辑器

  * 把这个镜像名字修改成为了一个错误的名字，比如：nginx:1.91。这样，这个 Deployment 就会出现一个升级失败的版本

    ```shell
    $ kubectl set image deployment/nginx-deployment nginx=nginx:1.91
    deployment.extensions/nginx-deployment image updated
    ```

  * 由于这个 nginx:1.91 镜像在 Docker Hub 中并不存在，所以这个 Deployment 的“滚动更新”被触发后，会立刻报错并停止

  * 来检查一下 ReplicaSet 的状态

    ```shell
    $ kubectl get rs
    NAME                          DESIRED   CURRENT   READY   AGE
    nginx-deployment-1764197365   2         2         2       24s
    nginx-deployment-3167673210   0         0         0       35s
    nginx-deployment-2156724341   2         2         0       7s
    ```

    * 新版本的 ReplicaSet（hash=2156724341）的“水平扩展”已经停止。而且此时，它已经创建了两个 Pod，但是它们都没有进入 READY 状态。这当然是因为这两个 Pod 都拉取不到有效的镜像
    * 与此同时，旧版本的 ReplicaSet（hash=1764197365）的“水平收缩”，也自动停止了。此时，已经有一个旧 Pod 被删除，还剩下两个旧 Pod

  * 如何让这个 Deployment 的 3 个 Pod，都回滚到以前的旧版本呢？

    * 只需要执行一条 kubectl rollout undo 命令，就能把整个 Deployment 回滚到上一个版本

      ```shell
      $ kubectl rollout undo deployment/nginx-deployment
      deployment.extensions/nginx-deployment
      ```

      * 在具体操作上，Deployment 的控制器，其实就是让这个旧 ReplicaSet（hash=1764197365）再次“扩展”成 3 个 Pod，而让新的 ReplicaSet（hash=2156724341）重新“收缩”到 0 个 Pod

  * 想回滚到更早之前的版本，要怎么办呢？

    * **首先，需要使用 kubectl rollout history 命令，查看每次 Deployment 变更对应的版本**

      * 而由于在创建这个 Deployment 的时候，指定了–record 参数，所以创建这些版本时执行的 kubectl 命令，都会被记录下来

        ```shell
        $ kubectl rollout history deployment/nginx-deployment
        deployments "nginx-deployment"
        REVISION    CHANGE-CAUSE
        1           kubectl create -f nginx-deployment.yaml --record
        2           kubectl edit deployment/nginx-deployment
        3           kubectl set image deployment/nginx-deployment nginx=nginx:1.91
        ```

      * 还可以通过这个 kubectl rollout history 指令，看到每个版本对应的 Deployment 的 API 对象的细节

        ```shell
        $ kubectl rollout history deployment/nginx-deployment --revision=2
        ```

    * **然后，就可以在 kubectl rollout undo 命令行最后，加上要回滚到的指定版本的版本号，就可以回滚到指定版本了**

      ```shell
      $ kubectl rollout undo deployment/nginx-deployment --to-revision=2
      deployment.extensions/nginx-deployment
      ```

      * 这样，Deployment Controller 还会按照“滚动更新”的方式，完成对 Deployment 的降级操作

  * 对 Deployment 进行的每一次更新操作，都会生成一个新的 ReplicaSet 对象，有些多余，甚至浪费资源

    * 所以，Kubernetes 项目还提供了一个指令，使得对 Deployment 的多次更新操作，最后 只生成一个 ReplicaSet

    * 具体的做法是，在更新 Deployment 前，要先执行一条 kubectl rollout pause 指令

      ```shell
      $ kubectl rollout pause deployment/nginx-deployment
      deployment.extensions/nginx-deployment paused
      ```

      * 这个 kubectl rollout pause 的作用，是让这个 Deployment 进入了一个“暂停”状态
      * 所以接下来，就可以随意使用 kubectl edit 或者 kubectl set image 指令，修改这个 Deployment 的内容了

    * 由于此时 Deployment 正处于“暂停”状态，所以对 Deployment 的所有修改，都不会触发新的“滚动更新”，也不会创建新的 ReplicaSet

    * 而等到对 Deployment 修改操作都完成之后，只需要再执行一条 kubectl rollout resume 指令，就可以把这个 Deployment“恢复”回来

      ```shell
      $ kubectl rollout resume deploy/nginx-deployment
      deployment.extensions/nginx-deployment resumed
      ```

    * 而在这个 kubectl rollout resume 指令执行之前，在 kubectl rollout pause 指令之后的这段时间里，对 Deployment 进行的所有修改，最后只会触发一次“滚动更新”

    * 可以通过检查 ReplicaSet 状态的变化，来验证一下 kubectl rollout pause 和 kubectl rollout resume 指令的执行效果

      ```shell
      $ kubectl get rs
      NAME               DESIRED   CURRENT   READY     AGE
      nginx-1764197365   0         0         0         2m
      nginx-3196763511   3         3         3         28s
      ```

      * 只有一个 hash=3196763511 的 ReplicaSet 被创建了出来

  * 不过，即使像上面这样小心翼翼地控制了 ReplicaSet 的生成数量，随着应用版本的不断增加，Kubernetes 中还是会为同一个 Deployment 保存很多很多不同的 ReplicaSet

  * 该如何控制这些“历史”ReplicaSet 的数量呢？

    * 很简单，Deployment 对象有一个字段，叫作 spec.revisionHistoryLimit，就是 Kubernetes 为 Deployment 保留的“历史版本”个数。所以，如果把它设置为 0，就再也不能做回滚操作了

*  Deployment 这个 Kubernetes 项目中最基本的编排控制器的实现原理和使用方法
  * Deployment 实际上是一个**两层控制器**。首先，它通过**ReplicaSet 的个数**来描述应用的版本；然后，它再通过**ReplicaSet 的属性**（比如 replicas 的值），来保证 Pod 的副本数量
  * Deployment 控制 ReplicaSet（版本），ReplicaSet 控制 Pod（副本数）
* Kubernetes 项目对 Deployment 的设计，实际上是代替完成了对“应用”的抽象，使得可以使用这个 Deployment 对象来描述应用，使用 kubectl rollout 命令控制应用的版本
* 可是，在实际使用场景中，应用发布的流程往往千差万别，也可能有很多的定制化需求。比如，应用可能有会话黏连（session sticky），这就意味着“滚动更新”的时候，哪个 Pod 能下线，是不能随便选择的
  * 这种场景，光靠 Deployment 自己就很难应对了。对于这种需求，“自定义控制器”，就可以帮助实现一个功能更加强大的 Deployment Controller

# 深入理解StatefulSet：拓扑状态

* Deployment 实际上并不足以覆盖所有的应用编排问题
  * 根本原因，在于 Deployment 对应用做了一个简单化假设
  * 它认为，一个应用的所有 Pod，是完全一样的。所以，它们互相之间没有顺序，也无所谓运行在哪台宿主机上。需要的时候，Deployment 就可以通过 Pod 模板创建新的 Pod；不需要的时候，Deployment 就可以“杀掉”任意一个 Pod
  * 但是，在实际的场景中，并不是所有的应用都可以满足这样的要求
    * 尤其是分布式应用，它的多个实例之间，往往有依赖关系，比如：主从关系、主备关系
    * 还有就是数据存储类应用，它的多个实例，往往都会在本地磁盘上保存一份数据。而这些实例一旦被杀掉，即便重建出来，实例与数据之间的对应关系也已经丢失，从而导致应用失败
  * 所以，这种实例之间有不对等关系，以及实例对外部数据有依赖关系的应用，就被称为“有状态应用”（Stateful Application）

* 得益于“控制器模式”的设计思想，Kubernetes 项目很早就在 Deployment 的基础上，扩展出了对“有状态应用”的初步支持。这个编排功能，就是：StatefulSet
* StatefulSet 的设计其实非常容易理解。它把真实世界里的应用状态，抽象为了两种情况：
  1. **拓扑状态**
     * 这种情况意味着，应用的多个实例之间不是完全对等的关系。这些应用实例，必须按照某些顺序启动，比如应用的主节点 A 要先于从节点 B 启动。而如果你把 A 和 B 两个 Pod 删除掉，它们再次被创建出来时也必须严格按照这个顺序才行。并且，新创建出来的 Pod，必须和原来 Pod 的网络标识一样，这样原先的访问者才能使用同样的方法，访问到这个新 Pod
  2. **存储状态**
     * 这种情况意味着，应用的多个实例分别绑定了不同的存储数据。对于这些应用实例来说，Pod A 第一次读取到的数据，和隔了十分钟之后再次读取到的数据，应该是同一份，哪怕在此期间 Pod A 被重新创建过。这种情况最典型的例子，就是一个数据库应用的多个存储实例
* 所以，**StatefulSet 的核心功能，就是通过某种方式记录这些状态，然后在 Pod 被重新创建时，能够为新 Pod 恢复这些状态**

## Headless Service

* 一个 Kubernetes 项目中非常实用的概念

* Service 是 Kubernetes 项目中用来将一组 Pod 暴露给外界访问的一种机制。比如，一个 Deployment 有 3 个 Pod，那么就可以定义一个 Service。然后，用户只要能访问到这个 Service，它就能访问到某个具体的 Pod

* 这个 Service 又是如何被访问的呢？

  * **第一种方式，是以 Service 的 VIP（Virtual IP，即：虚拟 IP）方式**
    * 比如：当访问 10.0.23.1 这个 Service 的 IP 地址时，10.0.23.1 其实就是一个 VIP，它会把请求转发到该 Service 所代理的某一个 Pod 上
  * **第二种方式，就是以 Service 的 DNS 方式**
    * 比如：这时候，只要访问“my-svc.my-namespace.svc.cluster.local”这条 DNS 记录，就可以访问到名叫 my-svc 的 Service 所代理的某一个 Pod
    * 第一种处理方法，是 Normal Service。这种情况下，访问“my-svc.my-namespace.svc.cluster.local”解析到的，正是 my-svc 这个 Service 的 VIP，后面的流程就跟 VIP 方式一致了
    * 而第二种处理方法，正是 Headless Service。这种情况下，访问“my-svc.my-namespace.svc.cluster.local”解析到的，直接就是 my-svc 代理的某一个 Pod 的 IP 地址。**可以看到，这里的区别在于，Headless Service 不需要分配一个 VIP，而是可以直接以 DNS 记录的方式解析出被代理 Pod 的 IP 地址**

*  Headless Service 的定义方式

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx
    labels:
      app: nginx
  spec:
    ports:
    - port: 80
      name: web
    clusterIP: None
    selector:
      app: nginx
  ```

  * 所谓的 Headless Service，其实仍是一个标准 Service 的 YAML 文件。只不过，它的 clusterIP 字段的值是：None，即：这个 Service，没有一个 VIP 作为“头”。这也就是 Headless 的含义。所以，这个 Service 被创建后并不会被分配一个 VIP，而是会以 DNS 记录的方式暴露出它所代理的 Pod

  * 而它所代理的 Pod，是Label Selector 机制选择出来的，即：所有携带了 app=nginx 标签的 Pod，都会被这个 Service 代理起来

  * 当按照这样的方式创建了一个 Headless Service 之后，它所代理的所有 Pod 的 IP 地址，都会被绑定一个这样格式的 DNS 记录

    ```shell
    <pod-name>.<svc-name>.<namespace>.svc.cluster.local
    ```

  * 这个 DNS 记录，正是 Kubernetes 项目为 Pod 分配的唯一的“可解析身份”（Resolvable Identity）
  * 有了这个“可解析身份”，只要知道了一个 Pod 的名字，以及它对应的 Service 的名字，就可以非常确定地通过这条 DNS 记录访问到 Pod 的 IP 地址

* StatefulSet 又是如何使用这个 DNS 记录来维持 Pod 的拓扑状态的呢？

  ````yaml
  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: web
  spec:
    serviceName: "nginx"
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
          image: nginx:1.9.1
          ports:
          - containerPort: 80
            name: web
  ````

  * serviceName=nginx 字段告诉 StatefulSet 控制器，在执行控制循环（Control Loop）的时候，请使用 nginx 这个 Headless Service 来保证 Pod 的“可解析身份”

  * 所以，当通过 kubectl create 创建了上面这个 Service 和 StatefulSet 之后，就会看到如下两个对象：

    ```shell
    $ kubectl create -f svc.yaml
    $ kubectl get service nginx
    NAME      TYPE         CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    nginx     ClusterIP    None         <none>        80/TCP    10s
     
    $ kubectl create -f statefulset.yaml
    $ kubectl get statefulset web
    NAME      DESIRED   CURRENT   AGE
    web       2         1         19s
    ```

  * 如果手比较快的话，还可以通过 kubectl 的 -w 参数，即：Watch 功能，实时查看 StatefulSet 创建两个有状态实例的过程

    * 如果手不够快的话，Pod 很快就创建完了。不过，你依然可以通过这个 StatefulSet 的 Events 看到这些信息

      ```shell
      $ kubectl get pods -w -l app=nginx
      NAME      READY     STATUS    RESTARTS   AGE
      web-0     0/1       Pending   0          0s
      web-0     0/1       Pending   0         0s
      web-0     0/1       ContainerCreating   0         0s
      web-0     1/1       Running   0         19s
      web-1     0/1       Pending   0         0s
      web-1     0/1       Pending   0         0s
      web-1     0/1       ContainerCreating   0         0s
      web-1     1/1       Running   0         20s
      ```

    * 通过上面这个 Pod 的创建过程，不难看到，StatefulSet 给它所管理的所有 Pod 的名字，进行了编号，编号规则是：-

    * 而且这些编号都是从 0 开始累加，与 StatefulSet 的每个 Pod 实例一一对应，绝不重复

    * 更重要的是，这些 Pod 的创建，也是严格按照编号顺序进行的。比如，在 web-0 进入到 Running 状态、并且细分状态（Conditions）成为 Ready 之前，web-1 会一直处于 Pending 状态

      * Ready 状态再一次提醒了我们，为 Pod 设置 livenessProbe 和 readinessProbe 的重要性

    * 当这两个 Pod 都进入了 Running 状态之后，你就可以查看到它们各自唯一的“网络身份”了

      * 使用 kubectl exec 命令进入到容器中查看它们的 hostname：

        ```shell
        # 可以看到，这两个 Pod 的 hostname 与 Pod 名字是一致的，都被分配了对应的编号
        $ kubectl exec web-0 -- sh -c 'hostname'
        web-0
        $ kubectl exec web-1 -- sh -c 'hostname'
        web-1
        ```

      * 试着以 DNS 的方式，访问一下这个 Headless Service

        ```shell
        $ kubectl run -i --tty --image busybox dns-test --restart=Never --rm /bin/sh 
        ```

        * 通过这条命令，启动了一个一次性的 Pod，因为–rm 意味着 Pod 退出后就会被删除掉。然后，在这个 Pod 的容器里面，尝试用 nslookup 命令，解析一下 Pod 对应的 Headless Service：

          ```shell
          $ kubectl run -i --tty --image busybox dns-test --restart=Never --rm /bin/sh
          $ nslookup web-0.nginx
          Server:    10.0.0.10
          Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local
           
          Name:      web-0.nginx
          Address 1: 10.244.1.7
           
          $ nslookup web-1.nginx
          Server:    10.0.0.10
          Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local
           
          Name:      web-1.nginx
          Address 1: 10.244.2.7
          ```

        * 从 nslookup 命令的输出结果中，可以看到，在访问 web-0.nginx 的时候，最后解析到的，正是 web-0 这个 Pod 的 IP 地址；而当访问 web-1.nginx 的时候，解析到的则是 web-1 的 IP 地址

        * 这时候，如果在另外一个 Terminal 里把这两个“有状态应用”的 Pod 删掉：

          ```shell
          $ kubectl delete pod -l app=nginx
          pod "web-0" deleted
          pod "web-1" deleted
          ```

        * 然后，再在当前 Terminal 里 Watch 一下这两个 Pod 的状态变化，就会发现一个有趣的现象：

          ```shell
          $ kubectl get pod -w -l app=nginx
          NAME      READY     STATUS              RESTARTS   AGE
          web-0     0/1       ContainerCreating   0          0s
          NAME      READY     STATUS    RESTARTS   AGE
          web-0     1/1       Running   0          2s
          web-1     0/1       Pending   0         0s
          web-1     0/1       ContainerCreating   0         0s
          web-1     1/1       Running   0         32s
          ```

          * 可以看到，当把这两个 Pod 删除之后，Kubernetes 会按照原先编号的顺序，创建出了两个新的 Pod。并且，Kubernetes 依然为它们分配了与原来相同的“网络身份”：web-0.nginx 和 web-1.nginx
          * 通过这种严格的对应规则，**StatefulSet 就保证了 Pod 网络标识的稳定性**
            * 比如，如果 web-0 是一个需要先启动的主节点，web-1 是一个后启动的从节点，那么只要这个 StatefulSet 不被删除，你访问 web-0.nginx 时始终都会落在主节点上，访问 web-1.nginx 时，则始终都会落在从节点上，这个关系绝对不会发生任何变化

        * 所以，如果再用 nslookup 命令，查看一下这个新 Pod 对应的 Headless Service 的话：

          ```shell
          $ kubectl run -i --tty --image busybox dns-test --restart=Never --rm /bin/sh 
          $ nslookup web-0.nginx
          Server:    10.0.0.10
          Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local
           
          Name:      web-0.nginx
          Address 1: 10.244.1.8
           
          $ nslookup web-1.nginx
          Server:    10.0.0.10
          Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local
           
          Name:      web-1.nginx
          Address 1: 10.244.2.8
          ```

          * 可以看到，在这个 StatefulSet 中，这两个新 Pod 的“网络标识”（比如：web-0.nginx 和 web-1.nginx），再次解析到了正确的 IP 地址（比如：web-0 Pod 的 IP 地址 10.244.1.8）
          * 通过这种方法，**Kubernetes 就成功地将 Pod 的拓扑状态（比如：哪个节点先启动，哪个节点后启动），按照 Pod 的“名字 + 编号”的方式固定了下来**。此外，Kubernetes 还为每一个 Pod 提供了一个固定并且唯一的访问入口，即：这个 Pod 对应的 DNS 记录
          * 这些状态，在 StatefulSet 的整个生命周期里都会保持不变，绝不会因为对应 Pod 的删除或者重新创建而失效
          * 不过，相信也已经注意到了，尽管 web-0.nginx 这条记录本身不会变，但它解析到的 Pod 的 IP 地址，并不是固定的。这就意味着，对于“有状态应用”实例的访问，必须使用 DNS 记录或者 hostname 的方式，而绝不应该直接访问这些 Pod 的 IP 地址

* StatefulSet 这个控制器的主要作用之一，就是使用 Pod 模板创建 Pod 的时候，对它们进行编号，并且按照编号顺序逐一完成创建工作。而当 StatefulSet 的“控制循环”发现 Pod 的“实际状态”与“期望状态”不一致，需要新建或者删除 Pod 进行“调谐”的时候，它会严格按照这些 Pod 编号的顺序，逐一完成这些操作

* 所以，StatefulSet 其实可以认为是对 Deployment 的改良

* 与此同时，通过 Headless Service 的方式，StatefulSet 为每个 Pod 创建了一个固定并且稳定的 DNS 记录，来作为它的访问入口

* 实际上，在部署“有状态应用”的时候，应用的每个实例拥有唯一并且稳定的“网络标识”，是一个非常重要的假设。

# 深入理解StatefulSet：存储状态

*  StatefulSet 对存储状态的管理机制，主要使用的是一个叫作 Persistent Volume Claim 的功能
* 要在一个 Pod 里声明 Volume，只要在 Pod 里加上 spec.volumes 字段即可。然后，就可以在这个字段里定义一个具体类型的 Volume 了，比如：hostPath
* 可是，有没有想过这样一个场景：**如果并不知道有哪些 Volume 类型可以用，要怎么办呢**？
* **Kubernetes 项目引入了一组叫作 Persistent Volume Claim（PVC）和 Persistent Volume（PV）的 API 对象，大大降低了用户声明和使用持久化 Volume 的门槛**

## PVC和PV

* 例如，有了 PVC 之后，一个开发人员想要使用一个 Volume，只需要简单的两步即可

  * **第一步：定义一个 PVC，声明想要的 Volume 的属性：**

    ```yaml
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: pv-claim
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
    ```

    * 在这个 PVC 对象里，不需要任何关于 Volume 细节的字段，只有描述性的属性和定义。比如，storage: 1Gi，表示我想要的 Volume 大小至少是 1 GiB；accessModes: ReadWriteOnce，表示这个 Volume 的挂载方式是可读写，并且只能被挂载在一个节点上而非被多个节点共享

  * **第二步：在应用的 Pod 中，声明使用这个 PVC：**

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: pv-pod
    spec:
      containers:
        - name: pv-container
          image: nginx
          ports:
            - containerPort: 80
              name: "http-server"
          volumeMounts:
            - mountPath: "/usr/share/nginx/html"
              name: pv-storage
      volumes:
        - name: pv-storage
          persistentVolumeClaim:
            claimName: pv-claim
    ```

    * 在这个 Pod 的 Volumes 定义中，只需要声明它的类型是 persistentVolumeClaim，然后指定 PVC 的名字，而完全不必关心 Volume 本身的定义

  * 这时候，只要创建这个 PVC 对象，Kubernetes 就会自动为它绑定一个符合条件的 Volume。可是，这些符合条件的 Volume 又是从哪里来的呢？

    * 答案是，它们来自于由运维人员维护的 PV（Persistent Volume）对象。看一个常见的 PV 对象的 YAML 文件：

      ```yaml
      kind: PersistentVolume
      apiVersion: v1
      metadata:
        name: pv-volume
        labels:
          type: local
      spec:
        capacity:
          storage: 10Gi
        rbd:
          monitors:
          - '10.16.154.78:6789'
          - '10.16.154.82:6789'
          - '10.16.154.83:6789'
          pool: kube
          image: foo
          fsType: ext4
          readOnly: true
          user: admin
          keyring: /etc/ceph/keyring
          imageformat: "2"
          imagefeatures: "layering"
      ```

      * 这个 PV 对象的 spec.rbd 字段，正是 Ceph RBD Volume 的详细定义。而且，它还声明了这个 PV 的容量是 10 GiB。这样，Kubernetes 就会为刚才创建的 PVC 对象绑定这个 PV

  * 所以，Kubernetes 中 PVC 和 PV 的设计，**实际上类似于“接口”和“实现”的思想**。开发者只要知道并会使用“接口”，即：PVC；而运维人员则负责给“接口”绑定具体的实现，即：PV

  * 这种解耦，就避免了因为向开发者暴露过多的存储系统细节而带来的隐患。此外，这种职责的分离，往往也意味着出现事故时可以更容易定位问题和明确责任，从而避免“扯皮”现象的出现

  * 而 PVC、PV 的设计，也使得 StatefulSet 对存储状态的管理成为了可能

    ```yaml
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: web
    spec:
      serviceName: "nginx"
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
            image: nginx:1.9.1
            ports:
            - containerPort: 80
              name: web
            volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
      volumeClaimTemplates:
      - metadata:
          name: www
        spec:
          accessModes:
          - ReadWriteOnce
          resources:
            requests:
              storage: 1Gi
    ```
    * 为这个 StatefulSet 额外添加了一个 volumeClaimTemplates 字段。从名字就可以看出来，它跟 Deployment 里 Pod 模板（PodTemplate）的作用类似。也就是说，凡是被这个 StatefulSet 管理的 Pod，都会声明一个对应的 PVC；而这个 PVC 的定义，就来自于 volumeClaimTemplates 这个模板字段。更重要的是，这个 PVC 的名字，会被分配一个与这个 Pod 完全一致的编号

    * 这个自动创建的 PVC，与 PV 绑定成功后，就会进入 Bound 状态，这就意味着这个 Pod 可以挂载并使用这个 PV 了

    * ：**PVC 其实就是一种特殊的 Volume**。只不过一个 PVC 具体是什么类型的 Volume，要在跟某个 PV 绑定之后才知道

    * 当然，PVC 与 PV 的绑定得以实现的前提是，运维人员已经在系统里创建好了符合条件的 PV（比如，在前面用到的 pv-volume）；或者，你的 Kubernetes 集群运行在公有云上，这样 Kubernetes 就会通过 Dynamic Provisioning 的方式，自动为你创建与 PVC 匹配的 PV

    * 所以，在使用 kubectl create 创建了 StatefulSet 之后，就会看到 Kubernetes 集群里出现了两个 PVC

      ```shell
      $ kubectl create -f statefulset.yaml
      $ kubectl get pvc -l app=nginx
      NAME        STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
      www-web-0   Bound     pvc-15c268c7-b507-11e6-932f-42010a800002   1Gi        RWO           48s
      www-web-1   Bound     pvc-15c79307-b507-11e6-932f-42010a800002   1Gi        RWO           48s
      ```

      * 可以看到，这些 PVC，都以“<PVC 名字 >-<StatefulSet 名字 >-< 编号 >”的方式命名，并且处于 Bound 状态

      * 这个 StatefulSet 创建出来的所有 Pod，都会声明使用编号的 PVC。比如，在名叫 web-0 的 Pod 的 volumes 字段，它会声明使用名叫 www-web-0 的 PVC，从而挂载到这个 PVC 所绑定的 PV

      * 使用如下所示的指令，在 Pod 的 Volume 目录里写入一个文件，来验证一下上述 Volume 的分配情况

        ```shell
        $ for i in 0 1; do kubectl exec web-$i -- sh -c 'echo hello $(hostname) > /usr/share/nginx/html/index.html'; done
        ```

        * 通过 kubectl exec 指令，我们在每个 Pod 的 Volume 目录里，写入了一个 index.html 文件。这个文件的内容，正是 Pod 的 hostname。比如，我们在 web-0 的 index.html 里写入的内容就是 "hello web-0"

        * 此时，如果在这个 Pod 容器里访问`“http://localhost”`，你实际访问到的就是 Pod 里 Nginx 服务器进程，而它会为你返回 /usr/share/nginx/html/index.html 里的内容。这个操作的执行方法如下所示：

          ```shell
          $ for i in 0 1; do kubectl exec -it web-$i -- curl localhost; done
          hello web-0
          hello web-1
          ```

      * 如果你使用 kubectl delete 命令删除这两个 Pod，这些 Volume 里的文件会不会丢失呢？

        ```shell
        $ kubectl delete pod -l app=nginx
        pod "web-0" deleted
        pod "web-1" deleted
        ```

        * 在被删除之后，这两个 Pod 会被按照编号的顺序被重新创建出来。而这时候，如果你在新创建的容器里通过访问`“http://localhost”`的方式去访问 web-0 里的 Nginx 服务：

          ```shell
          # 在被重新创建出来的 Pod 容器里访问 http://localhost
          $ kubectl exec -it web-0 -- curl localhost
          hello web-0
          ```

        * 就会发现，这个请求依然会返回：hello web-0。也就是说，原先与名叫 web-0 的 Pod 绑定的 PV，在这个 Pod 被重新创建之后，依然同新的名叫 web-0 的 Pod 绑定在了一起。对于 Pod web-1 来说，也是完全一样的情况

      * **这是怎么做到的呢？**

        * StatefulSet 控制器恢复这个 Pod 的过程
          * 首先，当你把一个 Pod，比如 web-0，删除之后，这个 Pod 对应的 PVC 和 PV，并不会被删除，而这个 Volume 里已经写入的数据，也依然会保存在远程存储服务里
          * 此时，StatefulSet 控制器发现，一个名叫 web-0 的 Pod 消失了。所以，控制器就会重新创建一个新的、名字还是叫作 web-0 的 Pod 来，“纠正”这个不一致的情况
          * 需要注意的是，在这个新的 Pod 对象的定义里，它声明使用的 PVC 的名字，还是叫作：www-web-0。这个 PVC 的定义，还是来自于 PVC 模板（volumeClaimTemplates），这是 StatefulSet 创建 Pod 的标准流程
          * 所以，在这个新的 web-0 Pod 被创建出来之后，Kubernetes 为它查找名叫 www-web-0 的 PVC 时，就会直接找到旧 Pod 遗留下来的同名的 PVC，进而找到跟这个 PVC 绑定在一起的 PV
          * 这样，新的 Pod 就可以挂载到旧 Pod 对应的那个 Volume，并且获取到保存在 Volume 里的数据
          * **通过这种方式，Kubernetes 的 StatefulSet 就实现了对应用存储状态的管理**

* StatefulSet 的工作原理

  * **首先，StatefulSet 的控制器直接管理的是 Pod**。这是因为，StatefulSet 里的不同 Pod 实例，不再像 ReplicaSet 中那样都是完全一样的，而是有了细微区别的。比如，每个 Pod 的 hostname、名字等都是不同的、携带了编号的。而 StatefulSet 区分这些实例的方式，就是通过在 Pod 的名字里加上事先约定好的编号
  * **其次，Kubernetes 通过 Headless Service，为这些有编号的 Pod，在 DNS 服务器中生成带有同样编号的 DNS 记录**。只要 StatefulSet 能够保证这些 Pod 名字里的编号不变，那么 Service 里类似于 web-0.nginx.default.svc.cluster.local 这样的 DNS 记录也就不会变，而这条记录解析出来的 Pod 的 IP 地址，则会随着后端 Pod 的删除和再创建而自动更新。这当然是 Service 机制本身的能力，不需要 StatefulSet 操心
  * **最后，StatefulSet 还为每一个 Pod 分配并创建一个同样编号的 PVC**。这样，Kubernetes 就可以通过 Persistent Volume 机制为这个 PVC 绑定上对应的 PV，从而保证了每一个 Pod 都拥有一个独立的 Volume
    * 在这种情况下，即使 Pod 被删除，它所对应的 PVC 和 PV 依然会保留下来。所以当这个 Pod 被重新创建出来之后，Kubernetes 会为它找到同样编号的 PVC，挂载这个 PVC 对应的 Volume，从而获取到以前保存在 Volume 里的数据

*  StatefulSet 的设计思想

  * StatefulSet 其实就是一种特殊的 Deployment，而其独特之处在于，它的每个 Pod 都被编号了。而且，这个编号会体现在 Pod 的名字和 hostname 等标识信息上，这不仅代表了 Pod 的创建顺序，也是 Pod 的重要网络标识（即：在整个集群里唯一的、可被的访问身份）
  * 有了这个编号后，StatefulSet 就使用 Kubernetes 里的两个标准功能：Headless Service 和 PV/PVC，实现了对 Pod 的拓扑状态和存储状态的维护

# 深入理解StatefulSet：有状态应用实践



