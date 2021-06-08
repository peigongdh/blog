## CI-CD定义

### CI

Continuous Integration (CI)
CI 的意思是 持续构建 。
负责拉取代码库中的代码后，执行用户预置定义好的操作脚本，通过一系列编译操作构建出一个 制品 ，并将制品推送至到制品库里面。
常用工具有 Gitlab CI，Github CI，Jenkins 等。
这个环节不参与部署，只负责构建代码，然后保存构建物。构建物被称为 制品，保存制品的地方被称为 “制品库”

### CD

CD 则有2层含义： 持续部署（Continuous Deployment） 和 持续交付（Continuous Delivery） 。
持续交付 的概念是：将制品库的制品拿出后，部署在测试环境 / 交付给客户提前测试。 
持续部署 则是将制品部署在生产环境。可以进行持续部署的工具也有很多： Ansible 批量部署， Docker 直接推拉镜像等等。当然也包括我们后面要写到的 Kubernetes 集群部署。

### 镜像库

字面意思，镜像库就是集中存放镜像的一个文件服务。镜像库在 CI/CD 中，又称 制品库 。
构建后的产物称为制品，制品则要放到制品库做中转和版本管理。常用平台有Nexus，Jfrog，Harbor或其他对象存储平台。
在这里，我们选用 Nexus3 作为自己的镜像库。因为其稳定，性能好，免费，部署方便，且支持类型多，是许多制品库的首选选型。

在 nexus 中，制品库一般分为以下三种类型：

- proxy: 此类型制品库原则上只下载，不允许用户推送。可以理解为缓存外网制品的制品库。例如，我们在拉取 nginx 镜像时，如果通过 proxy 类型的制品库，则它会去创建时配置好的外网 docker 镜像源拉取（有点像 cnpm ）到自己的制品库，然后给你。第二次拉取，则不会从外网下载。起到 内网缓存 的作用。
- hosted：此类型制品库和 proxy 相反，原则上 只允许用户推送，不允许缓存。这里只存放自己的私有镜像或制品。
- group：此类型制品库可以将以上两种类型的制品库组合起来。组合后只访问 group 类型制品库，就都可以访问。

docker 在推送一个镜像时，镜像的 Tag (名称:版本号) 开头必须带着镜像库的地址，才可以推送到指定的镜像库。例如 jenkins-test 是不能推送到镜像库的。而 172.16.81.7:8082/jenkins-test 则可以推送到镜像库。

### Kubernetes

Kubernetes 是 Google 开源的一个容器编排引擎，它支持自动化部署、大规模可伸缩、应用容器化管理。在生产环境中部署一个应用程序时，通常要部署该应用的多个实例以便对应用请求进行负载均衡。

通俗些讲，可以将 Kubernetes 看作是用来是一个部署镜像的平台。可以用来操作多台机器调度部署镜像，大大地降低了运维成本。
那么， Kubernetes 和 Docker 的关系又是怎样的呢？
一个形象的比喻：如果你将 docker 看作是飞机，那么 kubernetes 就是飞机场。在飞机场的加持下，飞机可以根据机场调度选择在合适的时间降落或起飞。
在 Kubernetes 中，可以使用集群来组织服务器的。集群中会存在一个 Master 节点，该节点是 Kubernetes 集群的控制节点，负责调度集群中其他服务器的资源。其他节点被称为 Node ， Node 可以是物理机也可以是虚拟机。

### Flannel

前面我们在配置文件中，有提到过配置Pod子网络，Flannel 主要的作用就是如此。
它的主要作用是通过创建一个虚拟网络，让不同节点下的服务有着全局唯一的IP地址，且服务之前可以互相访问和连接。

### Deployment

如果你将 k8s 看作是一个大型机场，那么 deployment 刚好就是机场内的停机坪。
根据飞机的种类进行划分停机坪，不同的停机坪都停着不同类型的飞机。只不过，deployment 要比停机坪还要灵活，随时可以根据剩余的空地大小（服务器剩余资源）和塔台的指令，增大/变小停机坪的空间。这个“增大变小停机坪空间的动作”，在k8s中就是 deployment 对它下面所属容器数量的扩容/缩小的操作。
那么这也就代表，deployment是无状态的，也就不会去负责停机坪中每架飞机之间的通信和组织关系。只需要根据塔台的指令，维护好飞机的更新和进出指令即可。这个根据指令维护飞机更新和进出的行为，在k8s中就是 deployment 对他下面的容器版本更新升级，暂停和恢复更新升级的动作。
在这里的容器，并不等于 Docker 中的容器。它在K8S中被称为 Pod 。那么 Pod 是什么 ?

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: front-v1
spec:
  selector:
    matchLabels:
      app: nginx-v1
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx-v1
    spec:
      containers:
      - name: nginx
        image: registry.cn-hangzhou.aliyuncs.com/janlay/k8s_test:v1
        ports:
        - containerPort: 80
```

### Pod

Pod 是 K8S 中最小的可调度单元（可操作/可部署单元），它里面可以包含1个或者多个 Docker 容器。在 Pod 内的所有 Docker 容器，都会共享同一个网络、存储卷、端口映射规则。一个 Pod 拥有一个 IP。
但这个 IP 会随着Pod的重启，创建，删除等跟着改变，所以不固定且不完全可靠。这也就是 Pod 的 IP 漂移问题。这个问题我们可以使用下面的 Service 去自动映射
我们经常会把 Pod 和 Docker 搞混，这两者的关系就像是豌豆和豌豆荚，Pod 是一个容器组，里面有很多容器，容器组内共享资源。

### service

deployment 是停机坪，那么 Service 则是一块停机坪的统一通信入口。它负责自动调度和组织deployment中 Pod 的服务访问。由于自动映射 Pod 的IP，同时也解决了 Pod 的IP漂移问题。
下面这张图就印证了 Service 的作用。流量会首先进入 VM（主机），随后进入 Service 中，接着 Service 再去将流量调度给匹配的 Pod 。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: front-service-v1
spec:
  selector:
    app: nginx-v1
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

### ingress

在前面，我们部署了 deployment 和 Service，实现了对服务的访问。
但是在实际使用中，我们还会根据请求路径前缀的匹配，权重，甚至根据 cookie/header 的值去访问不同的服务。为了达到这种负载均衡的效果，我们可以使用 k8s 的另一个组件 —— ingress
在日常开发中，我们经常会遇到路径分流问题。例如当我们访问 /a 时，需要返回A服务的页面。访问 /b，需要返回服务B的页面。这时候，我们就可以使用 k8s 中的 ingress 去实现。
在这里，我们选择 ingress-nginx。 ingress-nginx 是基于 nginx 的一个 ingress 实现。当然也可以实现正则匹配路径，流量转发，基于 cookie header 切分流量（灰度发布）。

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-demo
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - http:
      paths: 
       - path: /wss
         backend:
          serviceName: front-service-v1
          servicePort: 80
  backend:
     serviceName: front-service-v1
     servicePort: 80
```

### 灰度发布

Canary rules are evaluated in order of precedence. Precedence is as follows:  canary-by-header -> canary-by-cookie -> canary-weight
k8s 会优先去匹配 header ，如果未匹配则去匹配 cookie ，最后是 weight

```yaml
annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-by-cookie: "users_from_Beijing"
    nginx.ingress.kubernetes.io/canary-by-header: "test-header-key"
    nginx.ingress.kubernetes.io/canary-by-header-value: "test-header-value"
    nginx.ingress.kubernetes.io/canary-weight: "50"
```

### 滚动发布

就绪状态
- 第一步，我们准备一组服务器。这组服务器当前服务的版本是 V1。
- 接下来，我们将会使用滚动策略，将其发布到 V2 版本。
升级第一个 Pod
- 第二步开始升级。
- 首先，会增加一个 V2 版本的 Pod1 上来，将 V1 版本的 Pod1 下线但不移除。此时，V1版本的 Pod1 将不会接受流量进来，而是进入一个平滑期等待状态（大约几十秒）后才会被杀掉。
- 第一个 Pod 升级完毕
升级剩下的 Pod
- 与上同理，同样是新增新版本Pod后，将旧版本Pod下线进入平滑期但不删除。等平滑期度过后再删除Pod：


滚动发布作为众多发布类型的一种，必然也存在着一些优点和缺点：

优点
- 不需要停机更新，无感知平滑更新。
- 版本更新成本小。不需要新旧版本共存
缺点
- 更新时间长：每次只更新一个/多个镜像，需要频繁连续等待服务启动缓冲（详见下方平滑期介绍）
- 旧版本环境无法得到备份：始终只有一个环境存在
- 回滚版本异常痛苦：如果滚动发布到一半出了问题，回滚时需要使用同样的滚动策略回滚旧版本。

### Kubernetes 健康探针

存活探针	Pod 运行时	检测服务是否崩溃，是否需要重启服务	杀死 Pod 并重启
可用探针	Pod 运行时	检测服务是不是允许被访问到。	停止Pod的访问调度，不会被杀死重启
启动探针	Pod 运行时	检测服务是否启动成功	杀死 Pod 并重启

- initialDelaySeconds：容器初始化等待多少秒后才会触发探针。默认为0秒。
- periodSeconds：执行探测的时间间隔。默认10秒，最少1秒。
- timeoutSeconds：探测超时时间。默认1秒，最少1秒。
- successThreshold：探测失败后的最小连续成功数量。默认是1。
- failureThreshold：探测失败后的重试次数。默认是3次，最小是1次。

### Kubernetes Secret

Secret 是 Kubernetes 内的一种资源类型，可以用它来存放一些机密信息（密码，token，密钥等）。
信息被存入后，我们可以使用挂载卷的方式挂载进我们的 Pod 内。当然也可以存放docker私有镜像库的登录名和密码，用于拉取私有镜像。


### Kubernetes DNS

那么在 Kubernetes 中，如何做服务发现呢？我们前面写到过， Pod 的 IP 常常是漂移且不固定的，所以我们要使用 Service 这个神器来将它的访问入口固定住。
但是，我们在部署 Service 时，也不知道部署后的ip和端口如何。那么在 Kubernetes 中，我们可以利用 DNS 的机制给每个 Service 加一个内部的域名，指向其真实的IP。
在Kubernetes中，对 Service 的服务发现，是通过一种叫做 CoreDNS 的组件去实现的。
CoreDNS 是使用 Go 语言实现的一个DNS服务器。当然，它也不只是可以用在 Kubernetes 上。也可以用作日常 DNS 服务器使用。在 Kubernetes 1.11版本后，CoreDNS 已经被默认安装进了 Kubernetes 内。

在 Kubernetes DNS 里，服务发现规则有2种：跨 namespace 和同 namespace 的规则。
kubernetes namespace（命名空间）是 kubernetes 里比较重要的一个概念。 
在启动集群后，kubernetes 会分配一个默认命名空间，叫default。不同的命名空间可以实现资源隔离，服务隔离，甚至权限隔离。

### Kubernetes ConfigMap 

但是在日常开发部署时，我们还会遇到一些环境变量的配置：例如你的数据库地址，负载均衡要转发的服务地址等等信息。这部分内容使用 Secret 显然不合适，打包在镜像内耦合又太严重。
这里，我们可以借助 Kubernetes ConfigMap 来配置这项事情
ConfigMap 是 Kubernetes 的一种资源类型，我们可以使用它存放一些环境变量和配置文件。
信息存入后，我们可以使用挂载卷的方式挂载进我们的 Pod 内，也可以通过环境变量注入。和 Secret 类型最大的不同是，存在 ConfigMap 内的内容不会加密。

kubectl create configmap mysql-config \
--from-literal=host=demo-mysql-service \
--from-literal=port=30548 \
--from-literal=username=root \
--from-literal=database=demo-backend

curl 'http://127.0.0.1:30795/api/user/list' \
  -H 'Connection: keep-alive' \
  -H 'Cache-Control: max-age=0' \
  -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.141 Safari/537.36' \
  -H 'DNT: 1' \
  -H 'content-type: application/json' \
  -H 'Accept: */*' \
  -H 'Referer: http://8.133.162.129:30795/' \
  -H 'Accept-Language: zh-CN,zh;q=0.9,en;q=0.8' \
  --compressed \
  --insecure


## 学习心得

如果要让jenkins能够访问k8s的机器，需要加入同一个安全组