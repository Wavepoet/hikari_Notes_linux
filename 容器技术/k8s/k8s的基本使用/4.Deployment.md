# Deployment

k8s Deployment通过一个YAML文件，来对Pod进行声明式管理(更具YANL文化部创建、调度和维护Pod)，YAML文件里面定义了期望的pod状态，例如使用1.21版本的Nginx镜像运行3个Nginx程序的副本。Deployment则需要尽可能的使当前的实际状态和期望状态保持一致，如果实际状态发生了变化，Deployment会自动进行调整来恢复到期望状态。

Deployment用于**声明式管理无状态应用**的生命周期。本质上它是对 ReplicaSet 的更高层抽象，提供滚动更新、版本回滚、自愈等能力。

个人感觉Deployment和docker compose的的功能类似，都是用来编排容器的。但是Deployment更加强大，可以实现滚动升级、回滚、扩容等功能。

## Deployment的核心能力

一个 Deployment 对象能帮你搞定很多事情，其中最核心的有这几样：

- **自我修复 (Self-healing) / 故障转移**

  万一某个 Pod 挂了，或者它所在的服务器节点直接宕机，Deployment 就像一个尽职的管理员，会立刻发现副本数不够了。它会自动在其他健康的节点上创建一个新的 Pod 来“顶班”，确保你的应用实例数量始终不多不少，正好是你设定的那个数。

- **弹性伸缩 (Scaling)**

  当应用流量高峰来临，CPU、内存吃紧时，你可以手动或自动地增加 Pod 的数量来分担压力。Deployment 自身负责“执行”扩缩容操作，但“决策”通常由一个叫做 HPA (Horizontal Pod Autoscaler) 的组件来做。你设定一个规则（比如“CPU超过80%就加Pod”），HPA 就会监控资源，然后自动命令 Deployment 增减 Pod 数量。这让你的应用轻松应对流量波动。

- **版本回滚 (Rollback)**

  每次更新应用，Deployment 都会帮你记下历史版本。如果新发布的版本有 Bug，或者效果不理想，你可以用一条简单的命令，快速“撤销”这次发布，把应用恢复到之前的某个稳定版本，就像玩游戏的“读档”一样。

- **滚动更新 (Rolling Update)**

  这是 Deployment 最酷的功能之一。当你发布新版本时，它不会“一刀切”地杀掉所有旧 Pod，然后再启动新 Pod，那样会导致服务中断。相反，它会采用“滚动”的方式：先启动一个或几个新版 Pod，等它们正常运行后，再优雅地关闭一个或几个旧版 Pod，一步步地完成版本交替。整个过程平滑过渡，你的用户几乎感觉不到后台正在进行一场“大换血”。

## Deployment的使用

### 编写YAML文件

可以通过IDE的插件或者在线的工具来快速编写Deployment的YAML文件，这些工具基本都会有Deployment的模板。

YAML文件主要由基本属性，元数据，期望状态，Pod模板组成.通常的基本属性只有一个。而元数据，期望状态，Pod模板只有一个。

YAML文件例子：

```yaml
# --- 区块一：基本属性  ---
apiVersion: apps/v1 #API的版本
kind: Deployment # 类型

# --- 区块二：元数据 ---
metadata:
  name: frontend-app          # 1. Deployment 的名称
  namespace: default          # 2. 所在的命名空间，默认是 default
  labels:
    app: frontend             # 3. 给这个 Deployment 贴的标签
    env: production

# --- 区块三：期望状态  ---
spec:
  replicas: 3                 # 4. 期望维持 3 个 Pod 副本
  selector:                   # 5. 标签选择器
    matchLabels:
      app: frontend           # -> 必须和下方模板里的标签一模一样！
  strategy:                   # 6. 更新策略，可选，默认就是滚动更新
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%           # 升级时最多允许超出的 Pod 比例
      maxUnavailable: 25%     # 升级时最多允许不可用的 Pod 比例

# --- 区块四：Pod 模板 ---
  template:
    metadata:
      labels:                 # 7. 生产出来的 Pod 会自带这个标签
        app: frontend
    spec:
      containers:             # 8. 容器的具体定义
      - name: nginx-container # 容器名称
        image: nginx:1.21.6   # 镜像版本
        imagePullPolicy: IfNotPresent # 镜像拉取策略（本地有就不拉取）
        ports:
        - containerPort: 80   # 容器内部暴露的端口
        
        # 9. 资源配额
        resources:
          requests:           # 最少需要多少资源才能启动
            cpu: "100m"       # 100 millicpu (0.1个CPU核心)
            memory: "128Mi"   # 128 MB 内存
          limits:             # 最多允许使用多少资源（防止打爆宿主机）
            cpu: "500m"       # 最多 0.5 个CPU核心
            memory: "256Mi"
```

### 运行

提交YAML文件给k8s

```bash
kubectl apply -f frontend-deployment.yaml
```

可以使用2 ``kubectl --dry-run``来进行试运行。让系统导出一份标准的YAML文件。

```bash
kubectl create deployment my-app --image=nginx:latest --dry-run=client -o yaml > my-app.yaml
```

### 生命周期

查看所有的 Deployment

```bash
kubectl get deployment
```

关闭 Deployment

```bash
kubectl delete -f frontend-deployment.yaml
```
